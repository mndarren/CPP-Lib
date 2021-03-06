#PSQL Basic Commands
==========================
1. Install pgSQL: sudo apt-get install postgresql-9.6 <br>
		sudo apt-get install pgadmin3
2. Setting password<br>
	`sudo -U postgres psql postgres`
	`\password postgres`
	`\q`
3. Allowing local connection
	```
	/etc/postgresql/9.6/main/pg_hba.conf
	local   all   all     md5
	host    all   all   ::1/128 md5
	sudo service postgresql restart
	```
4. Allowing remote connection
	```
	/etc/postgresql/9.6/main/pg_hba.conf
	local   all   postgres   md5
	local   all   all     md5 (changed to the following)
	host    all   all   0.0.0.0/0   trust
	/etc/postgresql/9.6/main/postgresql.conf
	listen_addresses = '*'
	sudo service postgresql restart
	```
5. Querying JSON in Postgres
	```
	CREATE TABLE json_test (
	  id serial primary key,
	  data jsonb
	);
	INSERT INTO json_test (data) VALUES 
	  ('{}'),
	  ('{"a": 1}'),
	  ('{"a": 2, "b": ["c", "d"]}'),
	  ('{"a": 1, "b": {"c": "d", "e": true}}'),
	  ('{"b": 2}');
	SELECT * FROM json_test;
	SELECT * FROM json_test WHERE data = '{"a":1}';
	SELECT * FROM json_test WHERE data @> '{"a":1}';
	SELECT * FROM json_test WHERE data <@ '{"a":1}';
	SELECT * FROM json_test WHERE data ? 'a';
	SELECT * FROM json_test WHERE data ?| array['a', 'b'];
	SELECT * FROM json_test WHERE data ?& array['a', 'b'];
	SELECT * FROM json_test WHERE data ->> 'a' > '1';  --compare value (only int and str)
	SELECT * FROM json_test WHERE data -> 'b' > '1';   --comparison between primitives, objects and arrays
	--Give me objects where element b has a child object that has element c equal to the string "d".
	SELECT * FROM json_test WHERE data #> '{b,c}' = '"d"';
	SELECT * FROM json_test WHERE data #>> '{b,c}' = 'd';  --not JSON object comparison
	--a json null is different to an SQL NULL.
	INSERT INTO json_test (data) 
	VALUES ('[]'), ('[1,2,"a"]'), ('null'), ('1E7'), ('"abc"');
	SELECT * FROM json_test;
	SELECT * FROM json_test WHERE data ->> 'a' > '1'; **Error! for get operator**
	SELECT * FROM json_test WHERE data @> '{}' AND data ->> 'a' > '1';  --works
	SELECT * FROM json_test WHERE data @> '[]' AND data ->> 1 = '2';
	```
4. Tip for query something in this table not in another table<br/>
	one should always use LEFT JOIN / IS NULL or NOT EXISTS rather than NOT IN to find the missing values.
5. Character Type
	```
	character varying(n), varcahar(n)  -- variable length with limit
	character(n), char(n)  -- fixed-length, blank padded
	text  -- variable unlimited length
	1) if one explicitly casts a value to varchar(n) or char(n), over-length value will be truncated to n char without raising an error.
	2) char without length specifier => will be char(1)
	   varchar without length => will accept strings of any size.
	3) in fact character(n) is usually the slowest of the three because of its additional storage costs. In most situations text or character varying should be used instead.
	```
6. How to test roles
	```
	\du  --display all users/roles
	/* show privileges for the role */
	SELECT table_catalog, table_schema, table_name, privilege_type
	FROM   information_schema.table_privileges 
	WHERE  grantee = 'sw_developer_role';
	/* switch user */
	Set session authorization peter;
	/* show current user */
	SELECT SESSION_USER, CURRENT_USER;
	SET ROLE 'paul';
	SELECT SESSION_USER, CURRENT_USER;
	```
7. Show role privileges:
	```
	SELECT table_catalog, table_schema, table_name, privilege_type
	FROM   information_schema.table_privileges 
	WHERE  grantee = 'middleware_app_role';
	```
8. Show all tables: 
	\du <br/>
	SELECT * FROM pg_catalog.pg_tables;<br/>
9. export SQL
	```
	pg_dump -U username dbname > dbexport.pgsql
	```
10. How to make a copy of DB
	```
	CREATE DATABASE my_database_review WITH TEMPLATE my_database OWNER postgres;
	```
11. Check connections #
	```
	select usename from pg_stat_activity;
	SELECT sum(numbackends) FROM pg_stat_database;
	```

