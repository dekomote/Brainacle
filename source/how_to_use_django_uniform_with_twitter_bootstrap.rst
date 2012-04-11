How to use Django Uni-form with Twitter Bootstrap CSS
#####################################################

:date: 2012-02-13 20:58
:author: Dejan Noveski
:category: Coding
:tags: python, plugin, vim, coding, twitter, bootstrap, uniform


I'm sure, most of you, know what `django-uni-form <https://github.com/pydanny/django-uni-form>`_ is... For those that don't, django-uni-form is a 
very cool django app that makes form management/styling/layouting a breeze. You can do pretty much anything to a form with it: Split it in formsets, add
HTML between every element, wrap different elements in different containers etc. Also, it produces great HTML, and it manages the <form> tag, as well
as the submit/reset buttons. If you aren't using it, you should definitely start.

`Twitter Bootstrap <http://twitter.github.com/bootstrap/>`_ is a client side framework by twitter, encompassing many aspects - layout, sane default CSS, typography,
Javascript plugins, even some icons. It looks very professional, it's easy to use and very popular. The way bootstrap styles a form is what most people need when
building or prototyping a web app. But, you need to be able to change some css classes and html elements in order to work. 

Doing this with django-uni-form is extremely easy. All we need to do is change the default **field** template. I'll start with an example:


Bootstrap
=========

This step is easy. Just make sure you have downloaded the Bootstrap CSS file, and that it's properly loaded.


The HTML
========

Simple HTML is all we need:

.. code-block:: html

    {% load uni_form_tags %}
    <!doctype html>
    <html>
    <head>
      <title>Django Uniform and Bootstrap</title>      
      <!-- LOAD BOOTSTRAP CSS HERE -->
    </head>
    <body>      
      <header class="navbar">
      </header>
      <div role="main" class="container">
        <div class="row">
            <div class="span12">
                {% uni_form form form.helper %}
            </div>
        </div>
        <hr/>    
        <footer>
          Brainacle 2012
        </footer>
      </div>
      <!-- LOAD BOOTSTRAP JS HERE -->
    </body>
    </html>


The Form
========

I'm making a simple User edit ModelForm, with one extra text field:

.. code-block:: python

    from django import forms
    from django.contrib.auth.models import User
    from django.core.urlresolvers import reverse

    from uni_form.helper import FormHelper
    from uni_form.layout import Layout, ButtonHolder, Submit, Fieldset


    class UserEditForm(forms.ModelForm):
        bio = forms.CharField(label = _('Bio'), required = False, 
                        widget=forms.Textarea(attrs={'class':'span12'}))

        class Meta:
            model = User
            fields = ("username", "first_name", "last_name", "email")
        
        def __init__(self, *args, **kwargs):
            
            self.helper = FormHelper()
            self.helper.form_method = 'POST'
            self.helper.form_action = reverse("user_edit_view_name")
            self.helper.form_class = 'form-horizontal'

            self.helper.layout = Layout(
                Fieldset(
                    'Edit Account Info',
                    'username', 'first_name', 'last_name',
                    'email', 'bio', 'save',
                ),
                ButtonHolder(
                    Submit('save', 'Save', css_class='btn btn-large btn-primary pull-right')
                )
            )
            return super(UserEditForm, self).__init__(*args, **kwargs)

**Notes:** We add span10 class to the "bio" field textarea because labels will take 2 spans, and we want the textarea
fill the entire width. We add class "form-horizontal" to the form, to get the labels position in line with the inputs.
The button classes are also important.


The Magic
=========

Now, we have to tell uni-form to generate html that will be compatible with Bootstrap.
Lucky enough, uni-form provides us with the ability to override the generic field HTML template.
Just make a new file called **field.html** inside **[template root path]/uni_form/** folder with the
following content:

.. code-block:: html

    {% if field.is_hidden %}
        {{ field }}
    {% else %}
    <div class="control-group {% if field.errors %}error{% endif %}">
        <label for="{{ field.auto_id }}" class="control-label">
            {% if field.field.required %}<b>{% endif %}{{ field.label|safe }}{% if field.field.required %}*</b>{% endif %}
        </label>
        <div class="controls">
            {{ field }}
            {% if field.errors %}
                <span class="help-inline">{% for error in field.errors %}{{ error }}<br/> {% endfor %}</span>
            {% endif %}
            {% if field.help_text%}
                <p class="help-block">
                    {{ field.help_text|safe }}
                </p>
            {% endif %}
        </div>
    </div>
    {% endif %}

This template handles labels, required fields, help text, errors etc. The end result looks something like this:

.. container:: center-align

    .. image:: ../static/uploads/bootstrap_uniform.png

