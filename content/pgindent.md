Title: pgindent
Date: 2017-06-30 11:00
Modified: 2017-07-21 16:00
Category: PostgreSQL
Tags: PostgreSQL
Slug: pgindent
Authors: Alexey Chernyshov
Summary: New pgindent tool

The PostgreSQL code has really confusing indents. For the help for programmers,
there is a tool named pgindent located at `src/tools/pgindent`. Earlier it was
written in C and has had a set of dependencies, the installation was tedious.
Now, it is rewritten in Perl and downloads all dependencies itself. The only
requirement is `pg_config` on `PATH`, to run it in such way use `--build`
option. The tool clones git repo `pg_bsd_indent`, downloads typedefs, indents
and then deletes repo. If the previous run was not successful, the repository
will stay in the `src/tools/pgindent/pg_bsd_indent` and will cause an error for
git clone command, just remove it. The other point it generates files for
typedefs, do not forget to delete them before commit. To indent single file
run:
```
src/tools/pgindent/pgindent --build filename
```
To indent all files in current directory and subdirectories:
```
src/tools/pgindent/pgindent --build
```
There are also pgperltidy for Perl.
