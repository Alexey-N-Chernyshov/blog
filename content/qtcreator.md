Title: Qt Creator configuration for PostrgreSQL development
Date: 2017-06-25 16:00
Tags: Qt Creator, PostgreSQL
Summary: Qt Creator configuration for PostgreSQL development

PostgreSQL code is a lot about of macros, which hard to track. Qt Creator allows going to macros declaration on Ctrl+Click.

Initial configuration is extremely simple, just set tab width as 4 symbols.
The PostgreSQL code standard defines tab width as 4 symbols and uses a mix of tabs and spaces. It is awful. For example, the definition of a pointer is aligned with 3 spaces:
```
int     i;
int    *j;
```

The next step is to import project. Follow `New File or Project -> Import Project -> Import Existing Project` specify project name and location, and add to filters *.sql, *.sgml, and *.control. Qt Creator adds project files:

```
# Qt Creator files
postgresql.config
postgresql.creator
postgresql.files
postgresql.includes
*.autosave
*.creator.user # Stores per-project user settings
```
You do not want these files to be committed into repository either excluded with .gitignore file since they are specific to a particular workflow. Thus, the best approach is to add these files to `.git/info/exclude` file. This file has the same format as any `.gitignore`.

Ok, you are ready to work with PostgreSQL.
