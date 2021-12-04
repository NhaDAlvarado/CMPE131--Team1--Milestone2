# Track time spend on an assignment

### Track time function

This will track time spend on a task which was created in to-do list. Users will click "start" when they want to time themself, and click "stop" when they complete the task.

In routes.py, we added a start() funcion

```python
## Add Time spend on Assignment feature
@app_Obj.route ('/start/<int:id>', methods = ['GET', 'POST'])
def start(id):
    form = ToDoListForm()
    title = "Start Task"
    task = ToDoList.query.get_or_404(id)
    if request.method == 'POST':
        task.task_name = request.form['task_name']
        try:
            db.session.commit()
            return redirect ('/todolist')
        except:
            return flash('Error: could not update a task')
    else:
        return render_template('start.html', task = task, form=form,title=title)
```

### HTML page

We created a start.html to dislay a pop-up message to anounce how much time user spent.

```
{% extends "base.html" %} {% block content %}

<body class="text-center">
  <h1 style="color: black"><strong> {{task.task_name}} </strong></h1>
  <button onclick="start()">Start</button>
  <button onclick="stop()">Stop</button>
  <p style="color: black" id="timer1">0000</p>
  <script>
    var secondTime;
    var num = 0;
    secondTime = num;
    var stopSign = "F";
    var progress = setInterval(startTimer, 1000);
    function start() {
      progress();
    }
    function startTimer() {
      secondTime++;
      document.getElementById("timer1").innerHTML = secondTime;
    }

    function stop() {
      clearInterval(progress);
      var h = Math.floor(secondTime / 3600);
      var m = Math.floor((secondTime / 60) % 60);
      var s = Math.floor(secondTime % 60);
      var result = h + "hours" + m + "mins" + s + "seconds";
      alert("time spend on this task: " + result);
    }
  </script>

  {% endblock %}
</body>
```
