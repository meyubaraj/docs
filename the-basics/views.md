# Views

## Views

### Introduction

Views contain all the HTML that you’re application will use to render to the user. Unlike Django, views in Masonite are your HTML templates. All views are located inside the `resources/templates` directory.

All views are rendered with Jinja2 so we can use all the Jinja2 code you are used to. An example view looks like:

{% code title="resources/templates/helloworld.html" %}
```markup
<html>
  <body>
    <h1> Hello {{ 'world' }}</h1>
  </body>
</html>
```
{% endcode %}

### Creating Views

Since all views are located in `resources/templates`, we can use simply create all of our views manually here or use our `craft` command tool. To make a view just run:

{% code title="terminal" %}
```text
$ craft view hello
```
{% endcode %}

This will create a template under `resources/templates/hello.html`.

### Calling Views

The `View` class is loaded into the container so we can retrieve it in our controller methods like so:

{% code title="app/http/controllers/YourController.py" %}
```python
from masonite.view import View

def show(self, view: View):
    return view.render('dashboard')
```
{% endcode %}

This is exactly the same as using the helper function above. So if you choose to code more explicitly, the option is there for you.

{% hint style="success" %}
If this looks weird to you or you are not sure how the container integrates with Masonite, make sure you read the [Service Container](../architectural-concepts/service-container.md) documentation
{% endhint %}

#### Global Views

Some views may not reside in the `resources/templates` directory and may even reside in third party packages such as a dashboard package. We can locate these views by passing a `/` in front of our view.

For example as a use case we might pip install a package:

{% code title="terminal" %}
```text
$ pip install package-dashboard
```
{% endcode %}

and then be directed or required to return one of their views:

{% code title="app/http/controllers/YourController.py" %}
```python
def show(self, view: View):
    return view.render('/package/views/dashboard')
```
{% endcode %}

This will look inside the `dashboard.views` package for a `dashboard.html` file and return that. You can obviously pass in data as usual.

**Caveats**

It's important to note that if you are building a third party package that integrates with Masonite that you place any `.html` files inside a Python package instead of directly inside a module. For example, you should place .html files inside a file structure that looks like:

```python
package/
  views/
    __init__.py
    index.html
    dashboard.html
setup.py
MANIFEST.in
...
```

ensuring there is a `__init__.py` file. This is a Jinja2 limitation that says that all templates should be located in packages.

Accessing a global view such as:

{% code title="app/http/controllers/YourController.py" %}
```python
def show(self, view: View):
    return view.render('/package/dashboard')
```
{% endcode %}

will perform an absolute import for your Masonite project. For example it will locate:

```text
app/
config/
databases/
...
package/
  dashboard.html
```

Or find the package in a completely separate third part module. So if you are making a package for Masonite then keep this in mind of where you should put your templates.

### Passing Data to Views

Most of the time we’ll need to pass in data to our views. This data is passed in with a dictionary that contains a key which is the variable with the corresponding value. We can pass data to the function like so:

{% code title="app/http/controllers/YourController.py" %}
```python
def show(self, view: View, request: Request):
    return view.render('dashboard', {'id': request.param('id')})
```
{% endcode %}

{% hint style="info" %}
Remember that by passing in parameters like `Request` to the controller method, we can retrieve objects from the IOC container. Read more about the IOC container in the [Service Container](../architectural-concepts/service-container.md) documentation.
{% endhint %}

This will send a variable named `id` to the view which can then be rendered like:

{% code title="resources/templates/dashboard.html" %}
```markup
<html>
  <body>
    <h1> {{ id }} </h1>
  </body>
</html>
```
{% endcode %}

### View Syntax

Views use [Jinja2](http://jinja.pocoo.org/docs/2.10/) for it's template rendering. You can read about Jinja2 at the [official documentation here](http://jinja.pocoo.org/docs/2.10/).

Masonite also enables Jinja2 Line Statements by default which allows you to write syntax the normal way:

```ruby
{% extends 'nav/base.html' %}

{% block content %}
    {% for element in variables %}
        {{ element }}
    {% endfor %}

    {% if some_variable %}
        {{ some_variable }}
    {% endif %}

{% endblock %}
```

Or using line statements with the `@` character:

```ruby
@extends 'nav/base.html'

@block content
    @for element in variables
        {{ element }}
    @endfor

    @if some_variable
        {{ some_variable }}
    @endif

@endblock
```

The choice is yours on what you would like to use but keep in mind that line statements need to use only that line. Nothing can be after after or before the line.

### Adding Environments

{% hint style="success" %}
This section requires knowledge of [Service Providers](../architectural-concepts/service-providers.md) and how the [Service Container](../architectural-concepts/service-container.md) works. Be sure to read those documentation articles.
{% endhint %}

You can also add Jinja2 environments to the container which will be available for use in your views. This is typically done for third party packages such as Masonite Dashboard. You can extend views in a Service Provider in the boot method. Make sure the Service Provider has the `wsgi` attribute set to `False`. This way the specific Service Provider will not keep adding the environment on every request.

```python
from masonite.view import View

wsgi = False

...

def boot(self, view: View):
    view.add_environment('dashboard/templates')
```

By default the environment will be added using the PackageLoader Jinja2 loader but you can explicitly set which loader you want to use:

```python
from jinja2 import FileSystemLoader
from masonite.view import View
...
wsgi = False

...

def boot(self, view: View):
    view.add_environment('dashboard/templates', loader=FileSystemLoader)
```

The default loader of `PackageLoader` will work for most cases but if it doesn't work for your use case, you may need to change the Jinja2 loader type.

### Using Dot Notation

If using a `/` doesn't seem as clean to you, you can also optionally use dots:

```python
def show(self, view: View):
    view.render('dashboard.user.show')
```

if you want to use a global view you still need to use the first `/`:

```python
def show(self, view: View):
    view.render('/dashboard.user.show')
```

### Helpers

There are quite a few built in helpers in your views. Here is an extensive list of all view helpers:

#### Request

You can get the request class:

```markup
<p> Path: {{ request().path }} </p>
```

#### Static

You can get the location of static assets:

If you have a configuration file like this:

{% code title="config/storage.py" %}
```python
....
's3': {
  's3_client': 'sIS8shn...'
  ...
  'location': 'https://s3.us-east-2.amazonaws.com/bucket'
  },
....
```
{% endcode %}

```markup
...
<img src="{{ static('s3', 'profile.jpg') }}" alt="profile">
...
```

this will render:

```markup
<img src="https://s3.us-east-2.amazonaws.com/bucket/profile.jpg" alt="profile">
```

#### CSRF Field

You can create a CSRF token hidden field to be used with forms:

```markup
<form action="/some/url" method="POST">
    {{ csrf_field }}
    <input ..>
</form>
```

#### CSRF Token

You can get only the token that generates. This is useful for JS frontends where you need to pass a CSRF token to the backend for an AJAX call

```markup
<p> Token: {{ csrf_token }} </p>
```

#### Current User

You can also get the current authenticated user. This is the same as doing `request.user()`.

```markup
<p> User: {{ auth().email }} </p>
```

#### Request Method

On forms you can typically only have either a GET or a POST because of the nature of html. With Masonite you can use a helper to submit forms with PUT or DELETE

```markup
<form action="/some/url" method="POST">
    {{ request_method('PUT') }}
    <input ..>
</form>
```

This will now submit this form as a PUT request.

#### Route

You can get a route by it's name by using this method:

```markup
<form action="{{ route('route.name') }}" method="POST">
    ..
</form>
```

If your route contains variables you need to pass then you can supply a dictionary as the second argument.

```markup
<form action="{{ route('route.name', {'id': 1}) }}" method="POST">
    ..
</form>
```

or a list:

```markup
<form action="{{ route('route.name', [1]) }}" method="POST">
    ..
</form>
```

Another cool feature is that if the current route already contains the correct dictionary then you do not have to pass a second parameter. For example if you have a 2 routes like:

```python
Get('/dashboard/@id', 'Controller@show').name('dashboard.show'),
Get('/dashhboard/@id/users', 'Controller@users').name('dashhboard.users')
```

If you are accessing these 2 routes then the @id parameter will be stored on the user object. So instead of doing this:

```markup
<form action="{{ route('dashboard.users', {'id': 1}) }}" method="POST">
    ..
</form>
```

You can just leave it out completely since the `id` key is already stored on the request object:

```markup
<form action="{{ route('dashboard.users') }}" method="POST">
    ..
</form>
```

#### Back

This is useful for redirecting back to the previous page. If you supply this helper then the request.back\(\) method will go to this endpoint. It's typically good to use this to go back to a page after a form is submitted with errors:

```markup
<form action="/some/url" method="POST">
    {{ back(request().path) }}
</form>
```

Now when a form is submitted and you want to send the user back then in your controller you just have to do:

```python
def show(self, request: Request):
    # Some failed validation
    return request.back()
```

#### Old

The `request.back()` method will also flash the current inputs to the session so you can get them when you land back on your template. You can get these values by using the `old()` method:

```markup
<form>
  <input type="text" name="email" value="{{ old('email') }}">
  ...
</form>
```

#### Session

You can access the session here:

```markup
<p> Error: {{ session().get('error') }} </p>
```

{% hint style="success" %}
Learn more about session in the [Session](../advanced/sessions.md) documentation.
{% endhint %}

#### Sign

You can sign things using your secret token:

```markup
<p> Signed: {{ sign('token') }} </p>
```

#### Unsign

You can also unsign already signed string:

```markup
<p> Signed: {{ unsign('signed_token') }} </p>
```

#### Encrypt

This is just an alias for sign

#### Decrypt

This is just an alias for unsign

#### Config

This allows you to easily fetch configuration values in your templates:

```markup
<h2> App Name: {{ config('application.name') }}</h2>
```

#### Optional

Allows you to fetch values from objects that may or may not be None. Instead of doing something like:

```markup
{% if auth() and auth().name == 'Joe' %}
    <p>Hello!</p>
{% endif %}
```

You can use this helper:

```markup
{% if optional(auth()).name == 'Joe' %}
    <p>Hello!</p>
{% endif %}
```

#### DD

This is the normal dd helper you use in your controllers

#### Hidden

You can use this helper to quickly add a hidden field

```markup
<form action="/" method="POST">
    {{ hidden('secret' name='secret-value') }}
</form>
```

#### Exists

Check if a template exists

```markup
{% if exists('auth/base') %}
    {% extends 'auth/base.html' %}
{% else %}
    {% extends 'base.html' %}
{% endif %}
```

#### Cookie

Gets a cookie:

```markup
<h2> Token: {{ cookie('token') }}</h2>
```

#### Url

Get the URL to a location:

```markup
<form action="{{ url('/about', full=True) }}" method="POST">

</form>
```

## Jinja2

Below are some examples of the Jinja2 syntax which Masonite uses to build views.

### Line Statements

It's important to note that Jinja2 statements can be rewritten with line statements and line statements are **preferred** in Masonite. In comparison to Jinja2 line statements evaluate the whole line, thus the name line statement.

So Jinja2 syntax looks like this:

```markup
{% if expression %}
    <p>do something</p>
{% endif %}
```

This can be rewritten like this with line statement syntax:

```markup
@if expression
    <p>do something</p>
@endif
```

It's important to note though that these are line statements. Meaning nothing else can be on the line when doing these. For example you CANNOT do this:

```markup
<form action="@if expression: 'something' @endif">

</form>
```

But you could achieve that with the regular formatting:

```markup
<form action="{% if expression %} 'something' {% endif %}">

</form>
```

Whichever syntax you choose is up to you.

### Variables

You can show variable text by using `{{ }}` characters:

```markup
<p>
    {{ variable }}
</p>
<p>
    {{ 'hello world' }}
</p>
```

### If statement

If statements are similar to python but require an endif!

Line Statements:

```markup
@if expression
    <p>do something</p>
@elif expression
    <p>do something else</p>
@else
    <p>above all are false</p>
@endif
```

Using alternative Jinja2 syntax:

```markup
{% if expression %}
    <p>do something</p>
{% elif %}
    <p>do something else</p>
{% else %}
    <p>above all are false</p>
{% endif %}
```

### For Loops

For loop look similar to the regular python syntax.

Line Statements:

```markup
@for item in items
    <p>{{ item }}</p>
@endfor
```

Using alternative Jinja2 syntax:

```markup
{% for item in items %}
    <p>{{ item }}</p>
{% endfor %}
```

### Include statement

An include statement is useful for including other templates.

Line Statements:

```markup
@include 'components/errors.html'

<form action="/">

</form>
```

Using alternative Jinja2 syntax:

```markup
{% include 'components/errors.html' %}

<form action="/">

</form>
```

Any place you have repeating code you can break out and put it into an include template. These templates will have access to all variables in the current template.

### Extends

This is useful for having a child template extend a parent template. There can only be 1 extends per template:

Line Statements:

```markup
@extends 'components/base.html'

@block content
    <p> read below to find out what a block is </p>
@endblock
```

Using alternative Jinja2 syntax:

```markup
{% extends 'components/base.html' %}

{% block content %}
    <p> read below to find out what a block is </p>
{% endblock %}
```

### Blocks

Blocks are sections of code that can be used as placeholders for a parent template. These are only useful when used with the `extends` above. The "base.html" template is the parent template and contains blocks, which are defined in the child template "blocks.html".

Line Statements:

```markup
<!-- components/base.html -->
<html>
    <head>
        @block css 
        <!-- block named "css" defined in child template will be inserted here -->
        @endblock
    </head>

<body>
    <div class="container">
        @block content
        <!-- block named "content" defined in child template will be inserted here -->
        @endblock
    </div>

@block js
<!-- block named "js" defined in child template will be inserted here -->
@endblock

</body>
</html>
```

```markup
<!-- components/blocks.html -->
@extends 'components/base.html'

@block css
    <link rel=".." ..>
@endblock

@block content
    <p> This is content </p>
@endblock

@block js
    <script src=".." />
@endblock
```

Using alternative Jinja2 syntax:

```markup
<!-- components/base.html -->
<html>
    <head>
        {% block css %} 
        <!-- block named "css" defined in child template will be inserted here -->
        {% endblock %}
    </head>

<body>
    <div class="container">
        {% block content %} 
        <!-- block named "content" defined in child template will be inserted here -->
        {% endblock %}
    </div>

{% block js %}
<!-- block named "js" defined in child template will be inserted here -->
{% endblock %}

</body>
</html>
```

```markup
<!-- components/blocks.html -->
{% extends 'components/base.html' %}

{% block css %}
    <link rel=".." ..>
{% endblock %}

{% block content %}
    <p> This is content </p>
{% endblock %}

{% block js %}
    <script src=".." />
{% endblock %}
```

As you see blocks are fundamental and can be defined with Jinja2 and line statements. It allows you to structure your templates and have less repeating code.

The blocks defined in the child template will be passed to the parent template.

