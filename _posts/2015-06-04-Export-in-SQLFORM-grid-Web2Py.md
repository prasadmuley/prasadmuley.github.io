---
layout: post
title: Export in Web2Py's SQLFORM.grid
excerpt: "Useful hacks in Export functionality of Web2Py's SQLFORM.grid"
tags: [export, web2py, hack, SQLFORM, grid, javascript, button]
comments: true
---

[Web2Py's](http://web2py.com/) SQLFORM.grid is a very useful in complex CRUD operations. It provides several operations like create, update, delete, search, sort and browser records. It also gives pagination for large records. It is well explained here.

### Export functionality
* The export is one of the most useful functionalities of the SQLFORM.grid.
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

* You can use the add functionality of the grid to add some entries. Let's call the SQLFORM.grid in controllers/index.py

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
{% highlight html %}
{% raw %}
{% {{extend 'layout.html'}} %}
{% {{=grid}} %}
{% endraw %}
{% endhighlight %}

* Now check the index page.
{% highlight yaml %}
url: http://127.0.0.1:8000/sample_app/default/index
{% endhighlight %}

> Screenshot of Index page.
<figure style="display: block;">
	<a href="http://127.0.0.1:4000/images/sample_app_index.png"><img class="image_border" src="http://127.0.0.1:4000/images/sample_app_index.png"></a>
	<figcaption><a href="http://127.0.0.1:4000/images/sample_app_index.png">The SQLFORM.grid provides various export option like CSV, JSON, HTML etc</a>.</figcaption>
</figure>

* It shows various export type like CSV, JSON, HTML etc. But what if user wants to see only CSV format.

#### Export Classes

* All formats are displayed by default But we can decide whether to display all formats or single format like CSV.
* The exportclasses parameter decides which format of export will display or not.
* modify your code in controllers/default.py

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
    # show only CSV and disable other formats.
    export_classes = dict(csv=True, json=False, html=False,
                          tsv=False, xml=False, csv_with_hidden_cols=False,
                          tsv_with_hidden_cols=False)
    grid = SQLFORM.grid(query, showbuttontext=False,
                        exportclasses=export_classes)
    return dict(grid=grid)

{% endhighlight %}

> It shows only CSV format and doesn't show other formats.
<figure style="display: block;">
	<a href="http://127.0.0.1:4000/images/sample_app_only_csv.png"><img class="image_border" src="http://127.0.0.1:4000/images/sample_app_only_csv.png"></a>
	<figcaption><a href="http://127.0.0.1:4000/images/sample_app_only_csv.png">Export: CSV format is available only</a>.</figcaption>
</figure>

* Position of CSV button at the bottm of grid looks odd for user. What if user wants CSV button after search button.

#### Reposition Export button

* We can reposition the export buttons for all grids.
* add following javascript code in  views/layout.html. So changes are applicable on all grids.

{% highlight javascript %}
<script>
 $(document).ready(function() {
 // if export class exist then append it inside search form
   if($('.w2p_export_menu').length > 0) {
     var export_link = $('.w2p_export_menu a').clone();
	 //rename button to Export to CSV
	 export_link.text('Export to CSV');
	 $(export_link).appendTo('.web2py_console form')
     // remove web2py_export_menu
     $('.w2p_export_menu').remove();
 }
 });
</script>

{% endhighlight %}

> Save layout.html and refresh the index.html page.
<figure style="display: block;">
	<a href="http://127.0.0.1:4000/images/sample_app_export_to_CSV.png"><img class="image_border" src="http://127.0.0.1:4000/images/sample_app_export_to_CSV.png"></a>
	<figcaption><a href="http://127.0.0.1:4000/images/sample_app_export_to_CSV.png">Export to CSV button is available next to clear</a>.</figcaption>
</figure>
* Here,
    Web2Py adds all export links inside w2p_export_menu class.
We're cloning this class and appending cloned class to web2py_console form class.



* Let's export above data in CSV file. (Click on CSV button)

| employee.id |   employee.name   |          employee.email        | employee.company |
|:------------|:-----------------:|:------------------------------:|-----------------:|
|	  1		  |	 Tyrion Lannister |	       dwarf@google.com	       |       1          |
|---
|     2		  |  Jon Snow	      |       jon.snow@apple.com       |       3          |
|---
|     3	      | Jaime Lannister	  | jaime.lannister@microsoft.com  |	   2          |
|---
|     4	      | Stannis Lanister  |	  stannis.lanister@quora.com   |       4          |
|---
|     5	      |  Arya Starck	  |       arya.starck@quora.com	   |       4          |
|---
{: rules="groups"}

* Here,

   It shows company id not a company name in employee.company Because company is a foreign key in employee table.

So  How to display company name instead of ID in employee table?

##### Display name instead of id in export

*  We'll have to write a export class for CSV which will write name instead of id.
* It will work on all foreign keys even if table has multiple foreign keys

{% highlight python %}

class CSVExporter(object):
    """This class is used when grid's table contains foreign key () id of other table)
       Exported CSV should contain reference key name not id"""
    file_ext = "csv"
    content_type = "text/csv"

    def __init__(self, rows):
        self.rows = rows

    def export(self):
        if self.rows:
            s = StringIO()
            colnames = []
            for colname in self.rows.colnames:
                colnames.append(
                    colname.split('.')[-1].replace('_', ' ').upper())
            print colnames
            self.rows.export_to_csv_file(s, represent=True,
                                         colnames=colnames)
            return s.getvalue()
        else:
            return ''
{% endhighlight %}

* We'll have 