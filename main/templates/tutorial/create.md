{% raw %}

There are a few things that we need to do in order to start adding new
contacts. In short we're going to create a Form, a Handler and a Template.


### Contact Create Form

Create a new file `contact.py` in the `main` directory
and add the following code that will be responsible for validating the user's
input.

```python
from flask.ext import wtf
import wtforms

class ContactUpdateForm(wtf.Form):
  name = wtforms.StringField('Name', [wtforms.validators.required()])
  email = wtforms.StringField('Email', [wtforms.validators.optional(), wtforms.validators.email()])
  phone = wtforms.StringField('Phone', [wtforms.validators.optional()])
  address = wtforms.TextAreaField('Address', [wtforms.validators.optional()])
```

For more information regarding the form validation refert to
[Flask-WTForms](http://flask.pocoo.org/docs/patterns/wtforms/).


### Contact Create Handler

After creating the form we have to create a handler for it. Add the
following code into the `contact.py` file.

```python
import flask
import auth
import model
from main import app

@app.route('/contact/create/', methods=['GET', 'POST'])
@auth.login_required
def contact_create():
  form = ContactUpdateForm()
  if form.validate_on_submit():
    contact_db = model.Contact(
        user_key=auth.current_user_key(),
        name=form.name.data,
        email=form.email.data,
        phone=form.phone.data,
        address=form.address.data,
      )
    contact_db.put()
    return flask.redirect(flask.url_for('welcome'))
  return flask.render_template(
      'contact_create.html',
      html_class='contact-create',
      title='Create Contact',
      form=form,
    )
```

#### Let's take a closer look in this new snippet

```python
import flask
import auth
import model
from main import app
```

The necessary imports for creation handler, just put
them in the beginning of the `contact.py` file with the rest of the
imports.

```python
@app.route('/contact/create/', methods=['GET', 'POST'])
```

The route and the methods that we are going to use.
`GET` is to serve the html form and `POST` is to
submit the data.

For more information refer to Flask documentation on
[routing](http://flask.pocoo.org/docs/quickstart/#routing).

```python
@auth.login_required
```

This decorator's purpose is to make sure that who ever is entering
this URL will be already signed in so we could use the `user_key`
of the authenticated user. If the user is not logged in, she will be
redirected to the sign-in page and then back to this URL.

### Contact Create Template

After creating the form and a handler, we are going to need a template
to be able to get the user data. Create a new file
`contact_create.html` in the `templates` directory
and paste the following code there:

```html
# extends 'base.html'
# import 'macro/forms.html' as forms

# block content
  <div class="page-header">
    <h1>{{title}}</h1>
  </div>
  <div class="row">
    <div class="col-md-4">
      <form method="POST" action=".">
        <fieldset>
          {{form.csrf_token}}

          {{forms.text_field(form.name, autofocus=True)}}
          {{forms.email_field(form.email)}}
          {{forms.text_field(form.phone)}}
          {{forms.textarea_field(form.address)}}

          <button type="submit" class="btn btn-primary btn-block">
            Create Contact
          </button>
        </fieldset>
      </form>
    </div>
  </div>
# endblock
```


### Finalise Contact Creation

Before we can actually test the contact creation there are couple of things
that we have to complete.


#### Import contact.py

We will have to import the `contact.py` in the `main.py`,
because otherwise the Flask application won't be able to figure out the
routing rules. Include the following line bellow the rest of the imports
in the `main.py` file.

```python
import contact
```


#### Adding a link on the top bar

The most important thing to make the user experience better, we will have
to create a link for the users to be able to start adding new contacts
in their phonebook.
Add the lines `2 - 4` inside the `<ul class="nav">...</ul>` element that you
will find in the `header.html` file that is located in the `templates/bit`
directory.

```html
<ul class="nav navbar-nav">
  <li class="{{'active' if html_class == 'contact-create'}}">
    <a href="{{url_for('contact_create')}}"><i class="fa fa-file"></i> Create Contact</a>
  </li>
  ...
</ul>
```


### Testing Contact Creation

If your development server is still running, then by visting the
[http://localhost:8080/](http://localhost:8080/)
you should notice the Create Contact link on the top bar and you should be able
to actually start creating new contacts!

Create couple of new contacts and also try to add a contact without
a name, or try to enter an invalid email and then press on **Create Contact**
button.

Since the contact list is not visible in the application yet, you can visit the
Google App Engine's admin console to make sure that the contacts are being
added, by visiting the admin console:
[http://localhost:8081/datastore](http://localhost:8081/datastore?kind=Contact).

{% endraw %}
