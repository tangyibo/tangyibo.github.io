---
layout: post
title: 使用copy命令的二进制形式向PostgreSQL/Greenplum数据库批量导入数据|Greenplum 
category: Greenplum
tag: [Greenplum]
---

这篇文章主要介绍使用copy命令的二进制形式向PostgreSQL/Greenplum数据库批量导入数据。
本文分为以下几个部分：
1. copy命令语法
2. copy命令的二进制格式
3. COPY的二进制导入数据开发

# 使用copy命令的二进制形式向PostgreSQL/Greenplum数据库批量导入数据

Greenplum是面向数据仓库应用的关系型数据库，基于PostgreSQL开发，跟PostgreSQL的兼容性非常好。通常向PostgreSQL/Greenplum数据库中大批量写入数据，可以使用insert语句或者copy命令语句。

但经实际测试经验发现，对于PostgreSQL数据库来说，insert与copy的写入速度基本差不多（参考：[PG copy&insert性能对比](https://blog.csdn.net/jacicson1987/article/details/83383761)）；但对于Greenplum数据库来说，copy的写入速度要比insert快很多。

接下来，我们主要聊下PostgreSQL/Greenplum数据库的copy命令语句导入数据的用法及其开发。

## 一、copy命令语法

copy命令可以操作的文件类型有：txt、csv、二进制格式等。

```
COPY table_name [ ( column_name [, ...] ) ]
    FROM { 'filename' | PROGRAM 'command' | STDIN }
    [ [ WITH ] ( option [, ...] ) ]
 
COPY { table_name [ ( column_name [, ...] ) ] | ( query ) }
    TO { 'filename' | PROGRAM 'command' | STDOUT }
    [ [ WITH ] ( option [, ...] ) ]
 
where option can be one of:
 
    FORMAT format_name
    OIDS [ boolean ]
    FREEZE [ boolean ]
    DELIMITER 'delimiter_character'
    NULL 'null_string'
    HEADER [ boolean ]
    QUOTE 'quote_character'
    ESCAPE 'escape_character'
    FORCE_QUOTE { ( column_name [, ...] ) | * }
    FORCE_NOT_NULL ( column_name [, ...] )
    FORCE_NULL ( column_name [, ...] )
    ENCODING 'encoding_name'
```
其中option的设置的参数如下：

- FORMAT：指复制到文件的文件类型,如：CSV,TEXT。
- OIDS  ：指复制到文件时带上oid，但是当某个表没有oid时就会出错。
- FREEZE ：冻结数据，然后执行VACUUM FREEZE。
- DELIMITER：指在导出文件时的分隔符指定需要用单引号。在TEXT时默认为tab，CSV文件默认是逗号。不支持binary文件格式。
- HEADER：指在复制到文件时带上表字段名称。
- NULL：指定null值，默认为\N。
- ENCODING：指定文件的编码，如果没有指定就默认使用客户端的字符集。
- STDIN：指的是客户端程序的输入流。
- STDOUT: 声明输入前往客户端应用。
- BINARY: 使用二进制格式存储和读取，而不是以文本的方式。 在二进制模式下，不能声明 DELIMITERS，NULL 或者 CSV 选项。

## 二、copy命令的二进制格式

PostgreSQL的二进制copy文件格式可参考：https://www.postgresql.org/docs/9.4/sql-copy.html

### 1、使用copy的二进制的原因

当导入的数据中含有二进制类型的数据时，使用csv/text等文本方式的copy将会遇到错误。但二进制的copy不存在这种问题，虽然官方强调二进制的copy存在移植性的问题，但是我在用在程序导入数据的场景时，且可不必关心。

### 2、copy的二进制的文件格式

二进制格式的文件由一个文件头、数据（零个或多个元组）和文件尾构成。文件头和数据按网络字节顺序表示。如下：

**文件头**

文件头由15个字节的固定区域和一个变长的扩展区域组成。固定区域包括：

- 签名：11个字节的序列PGCOPY\nFF\r\n\0。
- 标志位：32位的整数。位的编号从0（最低位）到31（最高位）。当前只有一个标志位被使用，它是第16位，如果它的值是1，表示含有OID，如果它的值是0，表示不含OID。其它的标志位永远是0。
- 扩展区域：首先是一个32位的整数，表示扩展区域的长度，不包括这个32位的整数自身。当前，它的值是0。第一个元组紧跟在它的后面。

**元组**

每个元组以一个16位的整数开始，表示元组中域的个数。然后是元组中的每个域。每个域由一个32位的整数开始，它表示域的长度，后面跟着域的数据，-1表示域的值为空值。如果文件中包含OID，则它的值跟在元组的第一个16位整数的后面，而且就算组的域的个数时，OID不被计算在内，它也由一个32位的整数开始，表示OID的长度，当前，OID的长度固定是4字节，以后可能扩展到8字节。

**文件尾**

文件尾包含一个16位的整数，它的值是-1。

### 3、copy的二进制文件格式示例

![示例](https://img-blog.csdnimg.cn/20200303215747625.png)

上图是copy命令导出的二进制数据文件，图中用红色竖线做分隔符，把二进制文件分成不同的段。蓝色数字表示各段的编号，其中：

**文件头**包括第1段到第3段。第1段是11个字节的签名序列PGCOPY\nFF\r\n\0；第2段是标志位，占4个字节；第3段是扩展区域，至少4个字节，根据情况可能更多。

**元组**包括第4段到第18段第4、5、6、7、8段是第一条记录：第4段表示该元祖的列数，占2个字节；第5段表示第一列的长度，占4个字节；第6段是第一列的值；第7段是第二列的长度，占4个字节；第8段是第二列的值。第9、10、11、12、13段是第二条记录，同上。第14、15、16、17、18段是第三条记录，同上。

**文件尾**是第19段，占2个字节。

## 三、COPY的二进制导入数据开发

当搞清楚COPY的二进制文件格式后，进行数据导入开发将会变得容易。首先我们准备一个测试导入数据用的表：

```
CREATE TABLE "tang"."binary_data_table" (
	"id" int8 NOT NULL,
	"content" varchar(64),
	"binary" bytea,
	PRIMARY KEY ("id")
)
```

跟上上述定义的表结构，我们得出的copy命令的语句应为：

```
COPY "tang"."binary_data_table" ("id","content","binary") FROM STDIN BINARY
```

按照上述的二进制文件格式，使用Java语言调用PostgreSQL驱动中的API接口，如下：

```
import java.io.DataOutputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.util.ArrayList;
import java.util.List;
import org.postgresql.PGConnection;
import org.postgresql.copy.PGCopyOutputStream;
 
public class PGCopyOutputStreamTester {
 
	private static String getCopySql(String schemaName, String tableName, List<String> columns) {
		StringBuilder sb = new StringBuilder();
		sb.append("COPY \"");
		sb.append(schemaName);
		sb.append("\".\"");
		sb.append(tableName);
		sb.append("\" (\"");
		sb.append(String.join("\",\"", columns));
		sb.append("\") FROM STDIN BINARY");
		return sb.toString();
	}
 
	public static void main(String[] args) {
		String schemaName = "tang";
		String tableName = "binary_data_table";
		List<String> columnList = new ArrayList<String>();
		columnList.add("id");
		columnList.add("content");
		columnList.add("binary");
 
		try {
			// 首先与数据库建立连接
			Class.forName("org.postgresql.Driver");
			String url = "jdbc:postgresql://172.17.207.151:5432/study";
			Connection connection = DriverManager.getConnection(url, "study", "123456");
 
			// 根据schema、table及列拼接出copy语句
			String copySqlIn = getCopySql(schemaName, tableName, columnList);
			System.out.println(copySqlIn);
 
			// 基于PostgreSQL的驱动API构造向PG写入数据的输出流
			DataOutputStream os = new DataOutputStream(new PGCopyOutputStream((PGConnection) connection, copySqlIn));
			long start = System.currentTimeMillis();
 
			// 11 bytes 文件头部 签名字段
			byte[] headerSignature = { 'P', 'G', 'C', 'O', 'P', 'Y', '\n', (byte) 0xFF, '\r', '\n', '\0' };
			os.write(headerSignature);
			// 32 bit integer 文件头部标志位，无OID
			os.writeInt(0);
			// 32 bit header 文件头部扩展区域
			os.writeInt(0);
 
			// 元组列表
			for (int i = 0; i < 10000; ++i) {
 
				// 列的总个数，即为3
				os.writeShort(columnList.size());
 
				// 第1个字段id的值
				long field1 = 200L + i;
				os.writeInt(8);
				os.writeLong(field1);
 
				// 第2个字段content的值
				String field2 = "hello world two! :" + i;
				os.writeInt(field2.getBytes().length);
				os.write(field2.getBytes());
 
				// 第3个字段binary的值(模拟的二进制数据)
				String filed3 = "this is a test binray content";
				os.writeInt(filed3.getBytes().length);
				os.write(filed3.getBytes());
			}
 
			// 文件尾
			os.writeShort(-1);// 0xFFFF
 
			// 将流中的数据写入到网络中
			os.flush();
			os.close();
 
			long stoped = System.currentTimeMillis();
			System.out.println("Total copy record 10000, elipse:" + (stoped - start) / 1000.0F);
 
			// 关闭数据库连接
			connection.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
 
}
```

## 四、结束语

虽然在 SQL 标准里没有 COPY 语句，但是当你遇到PostgreSQL/Greenplum等类型的数据库时，会发现copy有它特别的用途。
