Title: Static blog with Pelican on GitHub Pages
Date: 2017-06-29 10:00
Category: misc
Tags: Pelican, blog
Slug: pelican-blog
Authors: Alexey Chernyshov
Summary: How to create blog with Pelican and deploy it on GitHub Pages

This [post](http://eax.me/pelican/) and circumstances make me a blogger. Here I
will share my first experience in a creation and deployment of a static blog. I
use [Pelican](http://docs.getpelican.com/en/stable/#) to generate the site and
[GitHub Pages](https://pages.github.com/) to host it.
# GitHub
GitHub distinguishes two [types of GitHub Pages]
(https://help.github.com/articles/user-organization-and-project-pages/):
 **User Pages** and **Project Pages**.

 - **User Page** provides URL as `https://<my-github-account>.github.io` and
requires site files to be on the `master` branch at the root. It creates
additional obstacles in a workflow.
 - **Project Page** URL is `https://<my-github-account>.github.io/<project>`.
The site can be deployed in separate branch `gh-pages` or `/docs` directory on
`master` branch. So, it is possible to use one repository both for the site
hosting and as a storage for the site generator scripts.

Eventually, my choice is User Page for my personal page with a link to the blog
and Project Page for the blog. So, I created two repositories:

 - `<my-github-account>.github.io` for [personal page]
(https://github.com/Alexey-N-Chernyshov/Alexey-N-Chernyshov.github.io) and
initialize it with the link to the blog.
 - [blog](https://github.com/Alexey-N-Chernyshov/blog) and initialize it with
README file.

We will consider only the second one, the blog. It is time to clone it from
GitHub.
```
$ git clone git@github.com:<github-account>/<github-account>.github.io.git
```
# Pelican
Pelican allows to write content in Markdown laguage and build pretty good HTML.
It is easy to use, has a lot of themes and plugins. And it is really easy to
start using it. Pelican is written in Python, thus to create a virtual
environment for the blog project will be a good idea.
[ghp-import](https://github.com/davisp/ghp-import) is a handy tool that will
help with the blog hosting. Finally, `pelican-quickstart` asks a set of
questions and creates a skeleton project.

```
$ mkvirtualenv pelican
$ pip install pelican markdown ghp-import
$ pelican-quickstart
Welcome to pelican-quickstart v3.7.1.

This script will help you create a new Pelican-based website.

Please answer the following questions so this script can generate the files
needed by Pelican.

    
> Where do you want to create your new web site? [.] .
> What will be the title of this web site? My blog
> Who will be the author of this web site? My name
> What will be the default language of this web site? [en] en
> Do you want to specify a URL prefix? e.g., http://example.com   (Y/n) y
> What is your URL prefix? (see above example; no trailing slash) http://<github-account>.github.io/blog
> Do you want to enable article pagination? (Y/n) y
> How many articles per page do you want? [10] 10
> What is your time zone? [Europe/Paris] 
> Do you want to generate a Fabfile/Makefile to automate generation and publishing? (Y/n) y
> Do you want an auto-reload & simpleHTTP script to assist with theme and site development? (Y/n) y
> Do you want to upload your website using FTP? (y/N) n
> Do you want to upload your website using SSH? (y/N) n
> Do you want to upload your website using Dropbox? (y/N) n
> Do you want to upload your website using S3? (y/N) n
> Do you want to upload your website using Rackspace Cloud Files? (y/N) n
> Do you want to upload your website using GitHub Pages? (y/N) y
> Is this your personal page (username.github.io)? (y/N) n
Done. Your new project is available at /path/to/the/blog/
```

Edit .gitignore and commit changes.
```
*.pid
*.pyc
*.swp
output/
```

`pelican-quickstart` creates two configuration files:

 - `pelicanconf.py` for local development
 - `publishconf.py` for product deployment

# Write a content
Ensure you work in virtual environment, if not run 
```
$ workon pelican
```
It is convenient to run development server on <https://localhost:8000> that
will display changes as you make them. You can start it in the new console to
see the output
```
$ make devserver
```
Note that `Ctrl+C` will not stop the server, use
```
$ ./develop_server.sh stop
```
[Pelican stores](http://docs.getpelican.com/en/stable/content.html) all content
in `content` directory. There are two types of content:

 - **pages** for content that do not change very often (e.g., About, Contacts)
 - **articles**, or chronological blog posts in our case

All content can be written in Markdown and should have metadata like this:
```
Title: My super title
Date: 2017-12-03 10:20
Modified: 2017-12-05 19:30
Category: Python
Tags: pelican, blog
Slug: my-super-post
Authors: Alexis Metaireau, Conan Doyle
Summary: Short version for index and feeds

This is the content of my super blog post.
```

You can start with about page in `content/pages/about.md`.

# Publishing
Once you have added content just run
```
$ make github
```
It will generate HTML, commit it to the `gh-pages` branch and pushes to GitHub.

# Google Analytics
Another good idea is to add Google Analytics to your blog. Go to
 <http://www.google.com/analytics/> and follow instructions. Add ID as 
`US-XXXXXXXX-X` to publishconf.py. Deploy as described above.

# Comments
Comments are provided by [Disqus](https://disqus.com). Head to the website and
create an account. Here you should copy website name which is equal to the
string before ```.disqus``` in URL of your account
(i.e. https://alexey-chernyshovs-blog.disqus.com)...
![alt text](images/disqus_website_name.png "Website name")
and paste to pelicanconf.py
```
DISQUS_SITENAME = 'alexey-chernyshovs-blog'
```
But since Disqus doesn’t recognize the local address, this setting should go
to publishconf.py.

# Further work
Themes will be considered further.
