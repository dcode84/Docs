# Django Documentation

## **Setup**

### **Download**

Django can be installed globally throughout the system, just visit and download it on [official Django](www.djangoproject.com/) or if **pip** is already installed on the machine, use the following command in the terminal

>**_pip install django_**

If needed, upgrade packages

### **Commands**

In the console, a django project can be created via
>**_django-admin startproject name_of_project_**

Now, after changing directory into this project folder, inside this django project an app/module can be created via 

>**_python manage.py startapp app_name_**

To run a development server, the following commands is used (make sure you are in the project directory)

>**_python manage.py runserver_**

## **Project Settings**

Depending on the requirements, some configurations might come in handy.

## **settings.py**
### **INSTALLED_APPS Dictionary**

To make sure django knows correctly about an app inside this project, it needs to be made aware of it by adding another line to the INSTALLED_APPS dictionary. Image a bookstore app, name bookstore, one of the following should work

>**'bookstore'** or **'bookstore.apps.BookstoreConfig'**

Remember that BookstoreConfig for the second choice is pointing to a class inside **bookstore/apps.py**\
It is important to note that it is possible that the apps need to be appended at the **end** of the INSTALLED_APPS dictionary

### **TEMPLATES Dictionary**

If template folders are placed and named correctly, django will know and find app template folders. For project wide templates, a setting needs to be set. Make sure that app template folders are always structured like 

>**project_folder/app_folder/templates/app_name/template_name.html**

and the global template folder resides inside the project folder itself.

It is important to have another folder called like the app name, because django would not be able to distinguish between identically named templates, even thought they are in different apps. Django looks inside template folders automatically, however when it injects templates, it does not include the full path. It starts from 

>**/templates/**

and ignores everything that is prepended. If another folder named the same as the app, django will be able to tell identically named templates from different apps inside the project apart.

For global templates, inside the TEMPLATES dictionary, inside the **'DIRS'** dictionary, the path to the global templates folder can be added like this **(potentially without the project_name, which depends on BASE_DIR and to what root folder it points)**

>**'DIRS': [\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BASE_DIR / "project_name" / "templates"\
]**

### **STATIC FILES**

For static files like CSS, JS and images to be picked up correctly, apps (for global static files the project) need to be extended with a **static** folder. While app static folders are picked up automatically, the same as templates apply there. Add another folder named the same as the app to the app static folder.

Global static folders need a configuration set, so django is made aware of it. In the settings.py where **STATIC_URL** already exists, below add another configuration named **STATICFILES_DIRS** like this (potentially the project name is not mandatory)

> **STATICFILES_DIRS = [\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BASE_DIR / "project_name" / "static"\
]**

# **URLS**

### **Project urls.py**

Inside the project urls.py, app urls need to be included via the include function from the **django.urls** module. It'll look like this

>**path("bookstore/", include("bookstore.urls"))**

### **App urls.py (without generic views)**

**The path function from django.urls aswell as the views.py need to be imported.**

Importing views is done via
> **from . import views**

Sometime it is important to add a variable above urlpatterns named **app_name**. The name of the app should be assigned to this. (e.g bookstore)

Urls can be given and should get a name. Here an example of an url with a dynamic parameter

> **path("book/<str:book_name>/", views.book, name='book')**

The book function inside the views module is used to process this url. The dynamic part of this url accepts a string. This can also be an int.

### **views.py**

Important modules will be (basic)

1. django.shortcuts 
    - render, get_object_or_404
2. django.http 
    - for what ever response you need
3. django.urls
    - reverse

The parameters inside the functions are important. First should always be **request**, the following parameters are url related. 

> **def book(request, book_name):**

From the example of the urls.py, we have a url with a dynamic parameter. These play together. If a dynamic parameter inside the url is named **book_name**, it is very important to give the parameter of the function inside the views module **the same name**.

#### **The render function**

> **render(HttpRequest, template_name, context)**

1. HttpRequest is the request passed to the function
2. template_name is the name and path to the template (html file)
3. context is used to provide the template any kind of data, so the template can use it

> [!NOTE] 
> Django looks inside the template folder automatically

>**return render(request, "bookstore/book.html", {\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"book_title": book_name,\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"book_attributes": book_attributes\
})**




