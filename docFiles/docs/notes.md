# Note taking function

### Require installation

```bash
pip install -r requirements.txt
```

### Add/Delete notes function

In routes.py, we created a view_note() function to let users view, add or delete a note. We also create notes_render() function to let user choose a file to upload, and display markdown version.

```python
#view, create, delete notes
@app_Obj.route('/view_notes')
def view_notes():
    return render_template ("dashboard.html")

#Note render
@app_Obj.route("/render", methods=['GET', 'POST'])
@login_required
def notes_render():
    html = ""
    form = noteForm()
    if form.validate_on_submit():
        text = form.file.data.read()
        # Reads Markdown and Displays as string
        newtext = str(text).replace("<p>b'",'').replace("'</p>",'')
        html = newtext[4:-3].split('\\n') #display a file as markdown format
    return render_template("notes_render.html", form=form, html = html)
```

### Updated forms.py

```python
class noteForm (FlaskForm):
    file = FileField('Choose file to upload')
    submit = SubmitField('Upload')
```

### Updated database

```python
class Note(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), unique=True)
    note = db.Column(db.String(1024), index=True)
    user_id = db.Column(db.Integer)

    def __repr__(self):
        return f'Note {self.title} : {self.note}'
```

### HTML page

In dashboard.html, users can add/delete a note, and they can also view their note.

```
{% extends "base.html" %} {% block content %}
<div class="note">
  <h2><strong>Note Taking</strong></h2>
  <div class="note-input">
    <h4>Add A New Note</h3>
    <div class="note-wrapper">
      <input type="text" id="note-title" placeholder="Title of your note" />
      <textarea
        id="note-content"
        placeholder="Write your note here...."
        rows="6"
      ></textarea>
      <button
        id="add-note-btn"
        class="btn btn-primary"
        type="button"
        data-toggle="modal"
        style="margin-top: 5px; margin-left: 25px; margin-bottom: 15px"
      >
        <span><i class="fas fa-plus"></i></span>
        Add Note
      </button>
    </div>
  </div>

  <div class="note-list"></div>

  <button
    type="button"
    class="btn btn-primary"
    id="delete-all-btn"
    data-toggle="modal"
    style="margin-top: 5px; margin-left: 25px; margin-bottom: 15px"
  >
    <span><i class="fas fa-trash"></i></span>
    Delete All
  </button>
</div>

<script>
  const noteListDiv = document.querySelector(".note-list");
  let noteID = 1;
  function Note(id, title, content) {
    this.id = id;
    this.title = title;
    this.content = content;
  }

  // Add eventListeners

  function eventListeners() {
    document.addEventListener("DOMContentLoaded", displayNotes);
    document
      .getElementById("add-note-btn")
      .addEventListener("click", addNewNote);

    noteListDiv.addEventListener("click", deleteNote);

    document
      .getElementById("delete-all-btn")
      .addEventListener("click", deleteAllNotes);
  }

  eventListeners();

  // get item from storage
  function getDataFromStorage() {
    return localStorage.getItem("notes")
      ? JSON.parse(localStorage.getItem("notes"))
      : [];
  }

  // add a new note in the list

  function addNewNote() {
    const noteTitle = document.getElementById("note-title");
    const noteContent = document.getElementById("note-content");

    if (validateInput(noteTitle, noteContent)) {
      let notes = getDataFromStorage();

      let noteItem = new Note(noteID, noteTitle.value, noteContent.value);
      noteID++;
      notes.push(noteItem);
      createNote(noteItem);

      // saving in the local storage

      localStorage.setItem("notes", JSON.stringify(notes));
      noteTitle.value = "";
      noteContent.value = "";
    }
  }
  //  input validation

  function validateInput(title, content) {
    if (title.value !== "" && content.value !== "") {
      return true;
    } else {
      if (title.value === "") title.classList.add("warning");
      if (content.value === "") content.classList.add("warning");
    }
    setTimeout(() => {
      title.classList.remove("warning");
      content.classList.remove("warning");
    }, 1600);
  }

  // create a new note div

  function createNote(noteItem) {
    const div = document.createElement("div");
    div.classList.add("note-item");
    div.setAttribute("data-id", noteItem.id);
    div.innerHTML = `
            <h3>${noteItem.title}</h3>
            <p>${noteItem.content}</p>
            <button type = "button" class = "btn delete-note-btn">
            <span><i class = "fas fa-trash" >
            Delete
            </buttton>
      `;
    noteListDiv.appendChild(div);
  }

  // display all the notes from the local storage

  function displayNotes() {
    let notes = getDataFromStorage();
    if (notes.length > 0) {
      noteID = notes[notes.length - 1].id;
      noteID++;
    } else {
      noteID = 1;
    }
    notes.forEach((item) => {
      createNote(item);
    });
  }

  // delete a note
  function deleteNote(e) {
    if (e.target.classList.contains("delete-note-btn")) {
      e.target.parentElement.remove();
      let divID = e.target.parentElement.dataset.id;
      let notes = getDataFromStorage();
      let newNotesList = notes.filter((item) => {
        return item.id !== parseInt(divID);
      });
      localStorage.setItem("notes", JSON.stringify(newNotesList));
    }
  }

  // delete all notes
  function deleteAllNotes() {
    localStorage.removeItem("notes");
    let noteList = document.querySelectorAll(".note-item");
    if (noteList.length > 0) {
      noteList.forEach((item) => {
        noteListDiv.removeChild(item);
      });
    }
    noteID = 1; //resetting noteID to 1
  }
</script>
{% endblock %}

```

In notes_render.html, users can upload their own note, then display in markdown format.

```
{% extends "base.html" %} {% block content %}

<form method="POST" enctype="multipart/form-data" novalidate>
  {{ form.csrf_token }}
  <p>
    {{ form.file.label }} <br />
    {{ form.file(size=32) }}
  </p>

  <p>{{ form.submit() }}</p>
</form>

{{ html }} {% for x in html %}
<p>{{ x }}</p>
{% endfor %} {% endblock %}

```
