---
layout: post
title: Export in Web2Py's SQLFORM.grid
excerpt: "Useful hacks in Export functionality of Web2Py's SQLFORM.grid"
tags: [export, web2py, hack, SQLFORM, grid, javascript, button]
comments: true
---

[Web2Py's](http://web2py.com/) SQLFORM.grid is a very useful in complex CRUD operations. It provides several operations like create, update, delete, search, sort and browser records. It also gives pagination for large records. It is well explained here.

### Export functionality
* One of the most useful functionalities of the SQLFORM.grid is the export.
* You can easily export your data in various format like CSV, JSON etc.
* The SQLFORM.grid supports following format:
   *  CSV
   *  CSV(hidden columns)
   *  HTML
   *  JSON
   *  TSV
   *  TSV(hidden columns)
   *  XML

I'll explain some useful hacks in this export functionality. Create a app called sample_app using appadmin or PyDAL's shell.
Let's create two tables in database first.
 
{% highlight python %}

db.define_table('company',
                Field('name', 'string', notnull=True),
                Field('address', 'text'),
                Field('abbreviation', 'string', length=3),
                format="%(name)s")

db.define_table('employee',
                Field('name', 'string', length=50, notnull=True),
                Field('email', 'string', notnull=True),
                Field('company', db.company),
                format="%(name)s")

{% endhighlight %}

* You'll have to add some entries for company and employee tables using appadmin or PyDAL shell.

* You can the add functionality of the grid. Let's modify code in controllers/index.py

{% highlight python %}

@auth.requires_login()
def index():
    """
    example action using the internationalization operator T and flash
    rendered by views/default/index.html or views/generic.html

    if you need a simple wiki simply replace the two lines below with:
    return auth.wiki()
    """
    query = (db.employee.id > 0)
    # pass query to SQLFORM.grid
    grid = SQLFORM.grid(query, showbuttontext=False)
    return dict(grid=grid)

{% endhighlight %}

* We'll have to modify the code in views/default/index.html

{% highlight jekyll %}

{{extend 'layout.html'}}
{{=grid}}

{% endhighlight %}

* Now check the index page.
{% highlight yaml %}
url: http://127.0.0.1:8000/sample_app/default/index
{% endhighlight %}
