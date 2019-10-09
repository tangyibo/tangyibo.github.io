---
layout: post
title: MySQL数据导入|MySQL数据库 
category: MySQL
tag: [MySQL]
---

这篇文章主要记录了一个用于导入MySQL数据库数据的常用脚本。
本文分为以下几个部分：
1. 脚本内容

## MySQL数据导入常用脚本整理
脚步内容如下：
> cat ./import_mysql.sh
'''
host=172.16.90.210
port=3306
user=tangyibo
pass=tangyibo
dbname=test_for_tang
pathdir=./2019-10-08_15-03-24

export PATH=/usr/local/whistle/mysql/bin:$PATH

for file in `ls $pathdir/*.sql`
do
        sql="
                use $dbname; 
                source $file
        "

        mysql -h $host -P $port -u$user -p$pass -e "$sql"
done

echo "done!"

'''