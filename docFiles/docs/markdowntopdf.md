# Convert a markdown note to pdf

### Convert function

In routes.py, we added notes_converttopdf() function.

```python
# Notes Markdown to Pdf
@app_Obj.route("/mark-to-pdf", methods=['GET', 'POST'])
@login_required
def notes_converttopdf():
    html = ""
    form = noteForm()
    if form.validate_on_submit():
        text = form.file.data.read()
        text = str(text).replace("<p>b'", '').replace("'</p>",'')    # Reads Markdown and Displays as string
        path_wkhtmltopdf = '/usr/local/bin/wkhtmltopdf'
        config_path = pdfkit.configuration(wkhtmltopdf=path_wkhtmltopdf)
        pdfkit.from_string(text, 'note.pdf', configuration=config_path)
        flash ('Your note is successfully converted to pdf')
    return render_template("notes_marktopdf.html", form=form)
```

### HTML page

```{% extends "base.html" %}
{% block content %}

<h3>Currently saves to Project Directory</h3>
<form method="POST" enctype = "multipart/form-data" novalidate>
{{ form.csrf_token }}
<p> {{ form.file.label }} <br>
{{ form.file(size=32) }}</p>

<p>{{ form.submit() }}</p>
</form>

{{ html }}

{% for x in html %}
    <p>{{ x }}</p>
{% endfor %}
{% endblock %}
```
