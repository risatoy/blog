---
layout: post
title:  "Creating Flask Calendar App using FullCalendar.js"
date:   2021-10-18 00:00:00 +0900
categories: Flask
---

This blog details the process of creating Calendar App in Flask using FullCalendar.js.


#### What You Will See At The End
![image](/blog/assets/images/2021101802.png)



### Setting Up Flask
##### 1. Create **app.py**
```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

##### 2. Create **index.html** in the **templates** folder
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Flask Calendar App</h1>
    <a href="/calendar">Calendar</a>
</body>
</html>
```

##### 3. Add route for calendar in **app.py**
```python
@app.route('/calendar')
def calendar():
    return render_template('calendar.html')
```

##### 4. Create **calendar.html** in **templates** folder
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
    
</body>
</html>
```

### Importing FullCalendar.js
##### 5. Include fullcalendar in **calendar.html**
```html
<head>
    (snip)
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/fullcalendar@5.5.0/main.min.css">
    <script src="https://cdn.jsdelivr.net/npm/fullcalendar@5.5.0/main.min.js"></script>
</head>
<body>
    <div id="calendar"></div>
    <script>
        let calendarEl = document.getElementById('calendar');

        let calendar = new FullCalendar.Calendar(calendarEl, {});

        calendar.render();
    </script>
</body>
```

##### 6. `flask run` to see if it works on local
You will see something like this
![image](/blog/assets/images/2021101801.png)

### Adding Events
##### 7. Add events list in **app.py**
```python
events = [
    {
        'title': 'event1',
        'date': '2021-10-15'
    },
    {
        'title': 'event2',
        'date': '2021-10-20'
    }
]

(snip)
@app.route('/calendar')
def calendar():
    return render_template('calendar.html', events=events)
```

##### 8. Update **calendar.html** to show the events on calendar
```html
let calendar = new FullCalendar.Calendar(calendarEl, {
    events: [
    {% for event in events %}
    {
        title: '{{ event.title }}',
        start: '{{ event.date }}',
    },
    {% endfor %}
    ]
});
```

##### 9. `flask run` again and you will see something like this!
![image](/blog/assets/images/2021101802.png)

##### Project Code
[https://github.com/risatoy/demo-flask-calendar](https://github.com/risatoy/demo-flask-calendar)