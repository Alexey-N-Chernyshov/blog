Title: pgindent
Date: 2017-06-30 11:00
Category: PostgreSQL
Tags: PostgreSQL
Slug: pgindent
Authors: Alexey Chernyshov
Summary: New pgindent tool

The PostgreSQL code has really confusing indents. For the help for programmers,
there is a tool named pgindent. Earlier it was written in C and has had a set
of dependencies, the installation was tedious. Now, it is rewritten in Perl and
don't require anything to run it.
To indent single file run:
```
src/tools/pgindent/pgindent filename
```
To indent all files in current directory:
```
src/tools/pgindent/pgindent
```
There are also pgperltidy for Perl.
