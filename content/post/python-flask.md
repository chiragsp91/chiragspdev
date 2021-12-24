---
title: "Setup a python flask application"
date: 2020-07-21
description: "How to setup a python flask application"
categories:
  - "Development"
tags: 
    - "Development"
    - "Flask"
    - "Python"
#math: false
lead: "How to setup a python flask application"
comments: true
mathjax: true
toc: false
#menu: main
---

{{< figure src="/images/flask-python.png" caption="">}}

Python being one of most used language in the world, its very east to develop web application using Python.

In Python you can basically develop web applications using,

1. Django
2. Flask

Django is mainly used for large applications and flask is mainly used for light weight applications. In this article we are going to cover the basics of Python flask.

Steps to setup a python flask application

1. Step 0 : Things you need already install.
    - Python 3:  You can use python 2 as well, but here we will be using python 3
    - Visual Studio Code : We will using visual studio code as our goti IDE. You can install it from here

        [Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)

2. Step 1:

    Now you need install flask. Open Visual Studio Code and open a new terminal. There type below command. This will install flask.

    ```powershell
    pip install flask
    ```

3. Step 2:

    Create a new .py file and import flask.

    ```python
    from flask import Flask , render_template , request
    ```

    We will be using only above mentioned items from overall flask import.

4. Step 3 :

Time to create the .html file. Create a new folder called 'templates' in the project root directory.

Name it as 'index.html'

In this html just input the below HTML code

```html
<h1>Hello World!</h1L
```

As you see from above code , we are gonna display 'Hello World!' in our our example Python application.

5. Step 4:

Now coming to the final step, all we need to do is to adding routing in python application for it to open 'index.html' file.

Routing is like an address for a web page.

So in your main python file, type below code,

```python
@app.route('/')
def index():
    return render_template("index.html")
```

And that's , it you have a default address for 'index.html' file. 

Now if you go to your terminal and type,

```powershell
flask run
```

The python application will be available. Open any browser and open

localhost:5000

Your application will be up and running!

Bonus Step :

If you want route 'index.html' to something else like 'localhost:5000/hello', in your main python file change the routing, as show below,

```powershell
app.route('/hello')
def index():
    return render_template("index.html")
```

Now if you stop flask application and rerun it, 'index.html' will be available at address 

localhost:5000/hello
