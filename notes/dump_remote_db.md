Dumping remote databases
========================


Mysql
-----

		$ ssh $USER@$servername.hzdesign.com '(  mysqldump -u$username -p$password $dbname )' > ${localfile:-./$dbname}.sql
		$ mysql -u$username -p$password $dbname < ${localfile:-./$dbname}.sql


Postgres
--------

		$ ssh $USER@$servername.hzdesign.com '(  pg_dump $dbname )' > ${localfile:-./$dbname}.sql;
		$ psql $dbname < ${localfile:-./$dbname}.sql
