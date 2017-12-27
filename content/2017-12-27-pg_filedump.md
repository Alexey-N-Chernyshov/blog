Title: Physical recovery with pg_filedump
Date: 2017-12-27 12:00
Tags: PostgreSQL
Summary: New features of pg\_filedump for recovery

# Physical recovery with pg\_filedump
If you can’t start your Postgres database and want to recover latest data from database heap files or want to recover just deleted or updated values, `pg_filedump` will help you.
## About
`pg_filedump` is a utility to dump contents of heap/index/control files. Some time ago it was enhanced to be suitable for physical recovery data from database heap files. Also recently there was added an ability to recover TOAST values and skip deleted values to the `pg_filedump` to be of full-value recovery utility.
### Behind the scene
[Tables in Postgres](https://www.postgresql.org/docs/current/static/storage-file-layout.html) are stored in heap files divided into segments which are gigabyte-sized by default. Segments consist of pages (8kb by default) which stores data row by row. If an attribute is too large, the TOAST mechanism takes place. Simply put, it compresses, slices data into chunks and stores them in the external table. When a transaction deletes some data, actually, it is not deleted from the file immediately and can be restored. That’s [how MVCC works](http://momjian.us/main/writings/pgsql/mvcc.pdf).

![alt text]({filename}/images/pg_heap_file_page.png)
## Usage
Let’s see the facilities of `pg_filedump`.
### Install
```
It’s easy to build.
git clone git://git.postgresql.org/git/pg_filedump.git
cd pg_filedump
make
```
### Example
Let’s create test table and populate it with data from psql. I added large text from files with size of couple KB with `pg_read_file` to demonstrate TOAST’ed data. Checkpoint at the end flushes data files to the disk:
```
# create table my_table(i int, t timestamp, content text);
# insert into my_table values (1, now(), 'some text');
# insert into my_table values (2, now(), ‘to be deleted’);
# insert into my_table values (3, now(), pg_read_file('file_to_delete.txt'));
# insert into my_table values (4, now(), pg_read_file('some_file.txt'));
# checkpoint;
```
We can get relation id in an easy way as (we’ll consider further the more general way to know the heap file name):
```
select relfilenode from pg_class where relname = 'my_table';
 relfilenode 
-------------
       16408
(1 row)
```
We can find it by name.
```
find /path/to/db/ -type f | grep 16408
```
And dump the file. We should pass types of the table with `-D` option.
```
./pg_filedump -D int,timestamp,text /path/to/database/base/12445/16408
*******************************************************************
* PostgreSQL File/Block Formatted Dump Utility - Version 10.0
*
* File: /var/lib/postgresql/9.6/main/base/12445/16408
* Options used: -D int,timestamp,text 
*
* Dump created on: Mon Dec 25 18:38:02 2017
*******************************************************************

Block    0 ********************************************************
<Header> -----
 Block Offset: 0x00000000         Offsets: Lower      40 (0x0028)
 Block: Size 8192  Version    4            Upper    7952 (0x1f10)
 LSN:  logid      0 recoff 0x0157e770      Special  8192 (0x2000)
 Items:    4                      Free Space: 7912
 Checksum: 0x0000  Prune XID: 0x0000027d  Flags: 0x0000 ()
 Length (including item array): 40

<Data> ------ 
 Item   1 -- Length:   50  Offset: 8136 (0x1fc8)  Flags: NORMAL
COPY: 1	2017-12-25 18:31:03.547059	some text
 Item   2 -- Length:   54  Offset: 8080 (0x1f90)  Flags: NORMAL
COPY: 2	2017-12-25 18:31:21.451178	to be deleted
 Item   3 -- Length:   58  Offset: 8016 (0x1f50)  Flags: NORMAL
COPY: 3	2017-12-25 18:32:00.895268	(TOASTED)
 Item   4 -- Length:   58  Offset: 7952 (0x1f10)  Flags: NORMAL
COPY: 4	2017-12-25 18:32:04.745484	(TOASTED)
```
The data we are interested in is after COPY. Option -o skips deleted data and -t option outputs TOAST’ed values.
```
./pg_filedump -o -D int,timestamp,text /var/lib/postgresql/9.6/main/base/12445/16408 | grep COPY

COPY: 1	2017-12-25 18:31:03.547059	some text
COPY: 4	2017-12-25 18:32:04.745484		very large string
```
But what if we don’t know the segment number or the database schema? For example, we cannot start Postgres instance. The Postgres stores all the data about tables in the table named `pg_class` with relfilenode id 1259. So, we can get the segment number by the name of our table. Here `~` in `-D` argument means the rest of the row we do not consider.
```
./pg_filedump -D name,oid,oid,oid,oid,oid,oid,~ /path/to/database/1259 | grep COPY | grep my_table
COPY: my_table	2200	16410	0	10	0	16408
```
Where the last number is our segment number 16408. We can obtain schema from the table name `pg_attribute` with relfilenode 1249 as well. The third column is an oid of the attribute type.
```
./pg_filedump -ot -D oid,name,oid,int,smallint,~ /var/lib/postgresql/9.6/main/base/12445/1249 | grep 16408
COPY: 16408	i	23	-1	4
COPY: 16408	t	1114	-1	8
COPY: 16408	content	25	-1	-1
COPY: 16408	ctid	27	0	6
COPY: 16408	xmin	28	0	4
COPY: 16408	cmin	29	0	4
COPY: 16408	xmax	28	0	4
COPY: 16408	cmax	29	0	4
COPY: 16408	tableoid	26	0	4
```
The next step is to get types by oids which are 23, 25 and 1114.
```
./pg_filedump -i -D name,~ /path/to/database/1247 | grep -A5 -E 'OID: (23|25|1114)'
  XMIN: 1  XMAX: 0  CID|XVAC: 0  OID: 23
  Block Id: 0  linp Index: 8   Attributes: 30   Size: 32
  infomask: 0x0909 (HASNULL|HASOID|XMIN_COMMITTED|XMAX_INVALID) 
  t_bits: [0]: 0xff [1]: 0xff [2]: 0xff [3]: 0x07 

COPY: int4
--
  XMIN: 1  XMAX: 0  CID|XVAC: 0  OID: 25
  Block Id: 0  linp Index: 10   Attributes: 30   Size: 32
  infomask: 0x0909 (HASNULL|HASOID|XMIN_COMMITTED|XMAX_INVALID) 
  t_bits: [0]: 0xff [1]: 0xff [2]: 0xff [3]: 0x07 

COPY: text
--
  XMIN: 1  XMAX: 0  CID|XVAC: 0  OID: 1114
  Block Id: 1  linp Index: 39   Attributes: 30   Size: 32
  infomask: 0x0909 (HASNULL|HASOID|XMIN_COMMITTED|XMAX_INVALID) 
  t_bits: [0]: 0xff [1]: 0xff [2]: 0xff [3]: 0x07 

COPY: timestamp
```
Here we know the relfilenode of the table we are looking for and the schema and can easily dump the content of the table.
```
./pg_filedump -D int,timestamp,text /path/to/database/base/16408
```
## Links
[Wiki page on pg\_filedump](https://wiki.postgresql.org/wiki/Pg_filedump)
[More on pg\_filedump. Find it interesting](https://blog.dbi-services.com/displaying-the-contents-of-a-postgresql-data-file-with-pg_filedump/)
[Slides about pg\_filedump on PG Day'17](https://afiskon.github.io/static/2017/pg-filedump-pgday2017.pdf)
[Article in Russian, part 1](https://habrahabr.ru/company/postgrespro/blog/319770/)
[Article in Russian, part 2](https://habrahabr.ru/company/postgrespro/blog/323644/)


