# Django Documentation

## Setup

#### Download

Django can be installed globally throughout the system, just visit and download it on [official Django](www.djangoproject.com/) or if **pip** is already installed on the machine, use the following command in the terminal

>_pip install django_

If needed, upgrade packages

#### Commands

In the console, a django project can be created via
>_django-admin startproject name_of_project_

Now, after changing directory into this project folder, inside this django project an app/module can be created via 

>_python manage\.py startapp app_name_

To run a development server, the following commands is used (make sure you are in the project directory)

>_python manage\.py runserver_

## Project Settings

Depending on the requirements, some configurations might come in handy.

### settings\.py
#### INSTALLED_APPS Dictionary

To make sure django knows correctly about an app inside this project, it needs to be made aware of it by adding another line to the INSTALLED_APPS dictionary. Image a bookstore app, name bookstore, one of the following should work

>'bookstore'** or **'bookstore.apps.BookstoreConfig'

Remember that BookstoreConfig for the second choice is pointing to a class inside **bookstore/apps.py**\
It is important to note that it is possible that the apps need to be appended at the **end** of the INSTALLED_APPS dictionary

#### TEMPLATES Dictionary

If template folders are placed and named correctly, django will know and find app template folders. For project wide templates, a setting needs to be set. Make sure that app template folders are always structured like 

>project_folder/app_folder/templates/app_name/template_name.html

and the global template folder resides inside the project folder itself.

It is important to have another folder called like the app name, because django would not be able to distinguish between identically named templates, even thought they are in different apps. Django looks inside template folders automatically, however when it injects templates, it does not include the full path. It starts from 

>/templates/

and ignores everything that is prepended. If another folder named the same as the app, django will be able to tell identically named templates from different apps inside the project apart.

For global templates, inside the TEMPLATES dictionary, inside the **'DIRS'** dictionary, the path to the global templates folder can be added like this **(potentially without the project_name, which depends on BASE_DIR and to what root folder it points)**

<pre>
'DIRS': [
    BASE_DIR / "project_name" / "templates"
]
</pre>

#### STATIC FILES

For static files like CSS, JS and images to be picked up correctly, apps (for global static files the project) need to be extended with a **static** folder. While app static folders are picked up automatically, the same as templates apply there. Add another folder named the same as the app to the app static folder.

Global static folders need a configuration set, so django is made aware of it. In the **settings\.py** where **STATIC_URL** already exists, below add another configuration named **STATICFILES_DIRS** like this (potentially the project name is not mandatory)

<pre>
STATICFILES_DIRS = [
    BASE_DIR / "project_name" / "static"
]
</pre>

# URLS

### Project urls\.py

Inside the project urls\.py, app urls need to be included via the include function from the **django.urls** module. It'll look like this

>path("bookstore/", include("bookstore.urls"))

### App urls\.py (without generic views)

**The path function from django.urls aswell as the views\.py need to be imported.**

Importing views is done via (dot means in the same folder)
> from . import views

Sometime it is important to add a variable above urlpatterns named **app_name**. The name of the app should be assigned to this. (e.g bookstore)

Urls can be given and should get a name. Here an example of an url with a dynamic parameter

> path("book/<str\:book_name>/", views.book, name='book')

The book function inside the views module is used to process this url. The dynamic part of this url accepts a string. This can also be an int.

> [!IMPORTANT]
> The **name** attribute is going to be important once templates are introduced. The name can then be used in correlation with the url-tag.

# Views

Logic for views (essentially controllers) will be defined in **views\.py** 

Important modules will be (basic)

1. django.shortcuts 
    - render, get_object_or_404
2. django.http 
    - for what ever response you need
3. django.urls
    - reverse

The parameters inside the functions are important. First should always be **request**, the following parameters are url related. 

> def book(request, book_name):

From the example of the **urls\.py**, we have a url with a dynamic parameter. These play together. If a dynamic parameter inside the url is named **book_name**, it is very important to give the parameter of the function inside the views module **the same name**.

#### The render function

> render(HttpRequest, template_name, context)

1. HttpRequest is the request passed to the function
2. template_name is the name and path to the template (html file)
3. context is used to provide the template any kind of data, so the template can use it

<pre>
return render(request, "bookstore/book.html", {
    "book_title": book_name,
    "book_attributes": book_attributes
})
</pre>

> [!NOTE] 
> Django looks inside the template folder automatically

## Class-based Views

### View

While the general View implementation is not much different from function based views, there is some slight differences.

#### get() and post()

It is possible to use general View as get aswell as post. There is no need to define separate classes, however, get() and post() need to be defined inside the view class. This works well with forms to keep them populated with data, if not all entries are correct, so the user does not have to put in all the data again.

<pre>
class Book(View):
    def get(self, request):
        ...
        return render(...)
    
    def post(self, request):
        ...
        return render(...)
</pre>

### TemplateView

TemplateView is a view type specificly designed to build views that render templates. Django will choose a specified template, once a GET request reaches this view and renders it with the contex containing parameters captured in the URL.

In this case, the TemplateView, rather the context, holds a dictionary of parameters that were automatically passed into the view. The books can then be queried with the gives kwargs to get a specific book. The context dictionary can then be filled with the book data. 

> [!NOTE]
> super().get_context_data() is used to get data from the parent class (TemplateView in the following example). It is a way to further extend views.

<pre>
class BookDetailView(TemplateView):
    template_name = "path_to_template.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        book = Book.objects.get(title=kwargs['book_name'])

        context['title'] = book.title

        return context
</pre>

To extend this view, a new view class is required, which then extends BookDetailView instead of TemplateView. That way, the new view holds all the context data from of BookDetailView

<pre>
class ExtendedBookDetailView(BookDetailView): ...
</pre>

### ListView

ListView is a class that builds upon TemplateView, but is specifically designed to work with list and helps to write much less code. The Base view takes the parameter ListView.

While it is possible to work with the context data, ListView preferably works with the model to fetch data. By default, the list object is accessable by the name **object_list** and properties can be accessed on that.

**context_object_name** is a built-in property to set the default name of the list object inside the template.

<pre>
class BooksView(ListView):
    template_name = "path_to_template.html"
    model = Book
    context_object_name = "books"
</pre>

There is many ways to manipulate the list object by overriding built-in functions or properties.

- paginate_by # property to set a maximum displayed amount
- get_ordering(self) # orders the list

### DetailView

For fetching a single piece of data via a model, a DetailView comes in handy. If extra data is needed, it is of cource possible to override get_context_data. Django automatically loads whatever model it is passed. To identify which entry of data it should fetch, it uses a slug or primary key that is supplied via the URL.

<pre>
class BooksView(ListView):
    template_name = "path_to_template.html"
    model = Book
</pre>

For the DetailView to work, it is important to alter the URL a bit. If it is supplied an <int\:id>, simply change it to <int\:pk> as pk stands for primary key. The value then is treated as a primary key. By default it will look at the id field inside the database.

Inside templates, the model is easily accessed by the modelname, all lowercase, in this case: "book"

### FormView

Instead of using the general View implementation and its get or post methods, the FormView is a much shorter version to work with.

# [TEMPLATES](https://docs.djangoproject.com/en/5.0/ref/templates/language/) (Django Template Language DTL)
## Template syntax

Inside django templates, any js, html and css functionality will work. Django translates the template into valid html.

## Variables {{ variable }} 

Double curly brackets are used to display a single value.

## Tags {% %}

Tags are more complex variables. They can be used to iterate over items or control the flow of the template. The for-tag is used to iterate over lists. Most tags are not self closing, so they have to be closed manually.

### for

<pre>
{% for person in people %}
    {{ person.name }}
{% endblock %} 
</pre>

### if 

Boolean operators (&&, ||, !) can be replaced by plain english (and, or, not) 

<pre>
{% if condition %}
    Do somthing
{% elif condition %}
    Do somthing
{% else %}
    Do somthing
{% endif %}
</pre>

### url

The url-tag is used to forward to a different link, so it is mainly used inside a href attribute of a html link-tag.

<pre>
{% url "urlname" value1, value2, ... %}
</pre>

### Filters {{ variable|filter }} 

There are many filters that can be applied to change a variable. The name of the variable depends on the context that is passed into the template. To name a few

1. variable|upper
2. variable|title
3. variable|length

## Template inheritance (Tags)

### block

Block is used by child templates to fill a base/layout template. It needs a name it can be referenced with.

> {% block name %}{% endblock %} 

### extends 

Extends tells the template engine that this template extends another. A child template defines this and then extends a base/layout template. Extends needs a path to the base/layout and does not need a closing tag.

> {% extends "path" %}

### include

Unlike other inheritance tags, include does not need a closing tag. However, it needs a path to the template that is to be included.

> {% include "path" %}

### load

Load provides a mechanism to load external files, such as static files like CSS, javascript or images.

> {% load static %}

This can then be used inside, for example link html tags. It is also possible to import certain modules of CSS files by providing the **from** keyword

> {% load foo bar **from** somelibrary %}

# Models

Django provides a models\.py file for every module. This is where we get into databases. By defining models such as Book, django will create tables from these models. In the case of Book, it will go ahead and create this table with a few twists. The table name will be pluralized and set to lowercase.

Inside these models, fields need to be defined. There is many types of fields, which will be columns in a relational database.

This is how to define a model in django
<pre>
class Book(models.Model)
    title = models.CharField(max_length=50)
    rating = models.IntegerField()
    description = models.CharField(max_lenght=255)
    date_published = models.DateField()
    author_id = models.ForeignKey(
        Author,
        on_delete=models.CASCADE
    )
</pre>

> [!NOTE]
> The variable name (such as title) will be the name of a column inside a table. FieldTypes are the corresponding data types inside the database.

## Creating an entry to the database

To create and object that will be passed into the database, simple contructor initialization can be used. So if a new book is published, an instance of the Book model can be created. Using the example of the previous section:

<pre>
    hunger_games = Book(
        title = "Hunger Games",
        rating = 5
        description = "Katnisss",
        author_id = 1
    )
</pre>

> [!NOTE]
> Some fields are initialized automatically, such as date_published - it will have the value of datetime.now

## Built-in functions

### save()

Calling save on a model object will the object data to the database.

### Model_name.objects

objects is a static field that exists for every model, because it inherits from the Model class.

#### .all()

Returns a QuerySet of all the items that are present in a specific table.

#### .get(**fields)

To get a specific item out of the database, get() can be used to get a unique entry. Throws an error if there is multiple entries.

#### .filter()

Filter is returning a QuerySet of all the items that meet a condition. Multiple conditions are allowed.

### Field Lookups



### Field Options

#### blank

If a field has blank=True, form validation will allow entry of an empty value. It is validation related.

#### editable

If False, the field will not be displayed in the admin or any other ModelForm. They are also skipped during model validation. Default is True.

#### null

If True, Django will store empty values as NULL in the database. Default is False.

## Field Types

#### CharField(max_length, **options)

CharField has a forced max_length and additional optional parameters. For larger texts, TextField is a better choice.

#### TextField(**options)

For unristricted texts, TextField is a more suitable choice. While it is possible to supply max_length, it is not recommended. Use CharField for restricted character lengths

#### IntegerField(**options)

An Integer. Values from -2147483648 to 2147483647 are safe in all databases supported by Django.

#### FloatField(**options)

A floating-point number represented in Python by a float instance. It is a less restricted version.

#### DecimalField(max_digits=None, decimal_places=None, **options)

A fixed-precision decimal number, represented in Python by a Decimal instance. 

##### max_digits 

Represents the maximum allowed number of digits. 

##### decimal_places 

Represents the amount of numbers after the floating point.

> [!IMPORTANT]
> Note that **max_digits** must be _greater than or equal to_ **decimal_places**. max_digits counts for numbers before and after the floating point.

#### DateField(auto_now=False, auto_now_add=False, **options)

A date, represented in Python by a datetime.date instance.

##### auto_now

Automatically set the field to now every time the object is saved. Useful for “last-modified” timestamps. Note that the current date is always used.

It is automatically updated when calling Model.save().

##### auto_now_add

Sets the field when the object is first created. Current date is always used.

> [!IMPORTANT]
> Setting auto_now or auto_now_add to True will cause the field to have editable=False and blank=True set. 
**VERY CONFUSING, BECAUSE MAKING THEM BASICALLY NULLABLE BY SETTING THEM TO FALSE MAKES MORE SENSE**

### Relationships

#### OneToMany

To set up a one to many relationship, it is mandatory to use **models.ForeignKey()** inside the model, which then points to another model.

<pre>
class Book(models.Model)
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE
    )
</pre>

A book will then only have _one_ author, but authors can be assigned to many books.

#### OneToOne

To set up a one-to-one relationship is very straight forward. On one of the models that are related, **models.OneToOneField()** is required. 

<pre>
class ISBN(models.Model)
    isbn = models.CharField(...)

class Book(models.Model)
    isbn = models.OneToOneField(
        ISBN,
        on_delete=models.CASCADE
    )
</pre>

> [!NOTE]
> Tables that are related by a one-to-one relationship can aswell be merged together from the start, unless there is a reason not to.

#### ManyToMany

Setting up a many-to-many relationship is just as simple by adding models.ManyToManyField() to one of the models. Django then creates a lookup table that links the tables.

<pre>
class Country(models.Model)
    name = models.CharField(...)
    code = models.CharField(...)

class Book(models.Model)
    published_countries = models.ManyToManyField(
        Country, 
        related_name="books"
    )
</pre>

# Forms

## Form Class

The form class is a simple way to define forms. Merely the shape with the different input fields are defined. This looks very similar to model fields. The form class has built-in validations. Unlike model fields, the form fields will not have any impact on the database per se.

<pre>
from django import forms

class BookForm(forms.Form):
    title = forms.CharField()
    published_countries = forms.ModelMultipleChoiceField(queryset=Country.objects.all(), widget=forms.CheckboxSelectMultiple)
</pre>

<pre>
def bookform(request):
    if request.method == 'POST':
        form = BookForm(request.POST)

        if form.is_valid():
            book = Book.objects.create(title=form.cleaned_data['title'],
            book.published_countries.set(form.cleaned_data['published_countries'])
            
            book.save()

            return HttpResponseRedirect("/bookstore/books")
    else:
        form = BookForm()
        
    return render(request, "bookstore/bookform.html", {
        "form": form
    })
</pre>

By first creating the book object with the create() method of Book, it is then possible to set the countries on the published_countries field. 

> [!NOTE]
> ManyToMany fields are not accessable during object initialization. 
When working with relationships, it may prove better to use ModelForm

## ModelForm Class

With ModelForms it is important to configure it with the nested Meta class. There the model it should work with is set, but also other configurations.

It is not required to create a new instance for new data, because .save() can already be called on the form. That is only possible by connecting the model with the form.

### Updating data

Since the ModelForm is already connected to to model, it may prove useful to fetch existing data from the database and use the form to update data. By creating an instance for the existing data and then passing that data to the form via the instance parameter, django passes existing data into their respective fields. This instance however, needs to be passed via the request from another endpoint.

<pre>
def bookform(request):
    if request.method == 'POST':
        existing_data = Book.objects.get(pk=request.pk)
        form = BookForm(request.POST, instance=existing_data)

        if form.is_valid():
            form.save()
            return HttpResponseRedirect("/bookstore/books")
    else:
        form = BookForm()
        
    return render(request, "bookstore/bookform.html", {
        "form": form
    })
</pre>

### Meta config

#### model

The model field hooks the model with the form itself. The model may not be instantiated.

#### fields

##### \_\_all\_\_

This special variable makes sure all fields are included in the form.

<pre>
fields = '__all__'
</pre>

##### only define specific fields

By defining specific fields inside the fields field, django makes sure to only include the specified fields.

<pre>
fields = [
    '...',
    '...'
]
</pre>

> [!IMPORTANT]
> By specifying fields, it is important to pass a list to fields.

#### exclude

With exlude it is possible to exclude specific fields from the form.

<pre>
exclude = [
    '...',
    '...'
]
</pre>

> [!IMPORTANT]
> By excluding fields, it is important to pass a list to exclude, even if only one field is excluded.

#### labels

ModelForm, by default, auto generates label based on property names. This is prevented by configuring the labels field. Property_name has to match the model property, while label_name is the label it should display.

<pre>
labels = {
    'property_name' : 'label_name',
    'property_name' : 'label_name'
}
</pre>

#### error_messages

By configuring error_messages, it is possible to configure error messages for fields and their respective cases, such as required or max_length

<pre>
error_messages = {
    'property_name' : {
        'required': 'error_message',
        'max_length': 'error_message'
    },
    '...' : {
        '...': '...'
    },
}
</pre>

# Migrations

Migrations are used to create or alter a database. Migrations are a set of instructions as python code on how to treat data or the database. These can be creating a table, altering a table or even deleting a table.

> python manage\.py makemigrations

This simply creates migrations, however it does not perform those operations yet. To make django perform all the migrations and run them against the database, a command called migrate is needed. However it will only run migrations that have not already been migrated.

> python managa\.py migrate

> [!IMPORTANT]
> - Do not seperate model attributes by comma, django will not pick up these fields.
> - Do not append _id on attribute names, as django will do it by convention.

# Query functions

With django, it is not needed to know SQL syntax. Django has built-in query functions, that are then executed on the database level. By calling query functions, django translates them into SQL syntax and executes it on the database.

> [!NOTE]
> It is important to note that django is smart enough to cache previous results and might not query the database again. Only when the same line is executed again.
> Django will only reach out to the database once save() is called on the objects, or the variable on which a query function is called on is further processed. 

<pre>
book = Book.objects.all() <- not hitting the database
print(book) <- hitting the database

book = Book(title="Blah", author="Pew")
book.save() <- hitting the database
</pre>

> [!WARNING]
> Even if it is not required to know SQL, it is highly recommended to know SQL nonetheless. SQL is a very crucial part of backend, aswell as fullstack development

#### List of common query functions

- order_by
- distinct
- all
- filter
- get
- aggregate
- exists

> [!IMPORTANT] 
> Some of them return a QuerySet, whilst others don't

## Cross Model Query - Reverse object related lookup

### _set

Having a look at the following setup

<pre>
class Author(models.Model):
    first_name = models.CharField(...)
    last_name = models.CharField(...)

class Book(models.Model)
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE
    )
</pre>

Book can easily be queried to the its corresponding author. This however proves difficult the other way around, since there is no foreign key setup for book in author.

The django built-in reverse lookup **_set** is a convenient way to get around this problem.

First of all, the author object, for which the related books should be found with, is needed. By convention, the reverse is then called on the author object by appending **_set** on book.

<pre>
author = Author.objects.get(first_name="whoever")
books = author.book_set.all()
</pre>

> [!NOTE]
> There is no book attribute on author, but django will split it apart and looks inside the book model, which does have a foreign key for author. Book is transformed to lowercase.

#### related_name

A little workaround _set is to define a related_name parameter to the field, that has the foreign key.

<pre>
class Book(models.Model)
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,
        related_name="books"
    )
</pre>

Django will then create a hidden field for books. That has the effect of not having to append **_set** on book anymore, but instead use the related name.

<pre>
author = Author.objects.get(first_name="whoever")
books = author.books.all()
</pre>