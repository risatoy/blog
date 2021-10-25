---
layout: post
title:  "Flask+MySQL app with Docker and docker-compose"
date:   2021-10-24 00:09:00 +0900
categories: Flask
customjs:
 - http://code.jquery.com/jquery-1.4.2.min.js
 - http://yourdomain.com/yourscript.js
---

This blog details the process of creating simple Flask with MySQL app using Docker and docker-compose.

#### What the app looks like at the end
![image](/blog/assets/images/2021102401.png)

#### Final File Strucutre
```
/
├── app
│   ├──templates
│   │   ├── index.html
│   │   └── users.html
│   ├── app.py
│   ├── db.yml
│   ├── Dockerfile
│   └── requirements.txt
├── db
│   └── init.sql
└── docker-compose.yml
```

#### 1. create **app** folder and add **app.py**

#### 2. create **db** folder and add **init.sql**
initialize the databse before the first time the app runs
```sql
CREATE DATABASE flaskapp;
use flaskapp;

CREATE TABLE users (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(50)
);

INSERT INTO users
    (name, email)
VALUES
    ('William', 'william@gmail.com'),
    ('Bentley', 'bentley@gmail.com');
```

#### 3. Create Dockerfile in app folder
```Dockerfile
FROM python:3.8

EXPOSE 5001

WORKDIR /app

COPY requirements.txt /app
RUN pip install -r requirements.txt

COPY * /app
CMD python app.py
```

#### 4. create requirements.txt inside of app folder
```
Flask
flask-mysqldb
pyyaml
```

### Creating docker-compose
#### 5. create docker-compose.yml and app service
```yml
version: "3"
services:
    build: ./app
    volumes:
      - "./app:/app"
    links:
      - db
    ports:
      - "5001:5001"
```
build: specifies the directory which contains the Dockerfile containing the instructions for building this service
volumes: sync local folder to container folder
links: links this service to another container. This will also allow us to use the name of the service instead of having to find the ip of the database container, and express a dependency which will determine the order of start up of the container
ports: mapping of Host:Container ports

#### 6. add db service to docker-compose.yml
```yml
version: "3"
services:
    app:
    (...)
    db:
        image: mysql:5.7
        environment:
            MYSQL_ROOT_PASSWORD: root
        volumes:
            - ./db:/docker-entrypoint-initdb.d/:ro
```
environment: add environment variables. The specified variable is required for this image, and as its name suggests, configures the password for the root user of MySQL in this container. More variables are specified here.
volumes: since we want the container to be initialized with our schema, we wire the directory containing our init.sql script to the entry point for this container, which by the image’s specification runs all .sql scripts in the given directory.

### Setting up Flask app
#### 7. add following code to app.py to connect to database
```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

#### 8. create templates folder inside app folder, and add index.html
```html
<form method="POST">
    Name <input type="text" name="name" />
    <br>
    Email <input type="email" name="email" />
    <br>
    <input type="submit"> 
</form>
```

#### 9. create configuration file **db.yml** inside of app folder to store info for database
```yml
mysql_host: 'db'   # the service name used in docker-compose.yml
mysql_user: 'root'
mysql_password: 'yourpassword'
mysql_db: 'flaskapp'

```


#### 10. add following code in app.py to connect to database
```python
from flask import Flask, render_template
from flask_mysqldb import MySQL  # ADD THIS LINE

(...)

# Congigure db
with open('db.yml') as file:
    db = yaml.safe_load(file)
    app.config['MYSQL_HOST'] = db['mysql_host']
    app.config['MYSQL_USER'] =  db['mysql_user']
    app.config['MYSQL_PASSWORD'] =  db['mysql_password']
    app.config['MYSQL_DB'] =  db['mysql_db']

mysql = MySQL(app)

@app.route('/')
(...)

```

#### 11. add following code in app.py to fetch data from from
```python
from flask import Flask, render_template, request # ADD request
(...)
import yaml  # ADD THIS LINE 

(...)

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method =='POST':
        # Fetch form data
        userDetails = request.form
        name = userDetails['name']
        email = userDetails['email']
        cur = mysql.connection.cursor()
        cur.execute("INSERT INTO users(name, email) VALUES(%s, %s)", (name, email))
        mysql.connection.commit()
        cur.close()
        return 'success'
    return render_template('index.html')

(...)
```
#### 12. execute `docker-compose build` and `docker-compose up` > on localhost:5001, check if you will get 'success' screen after submitting name and email 

#### 13. now add a page to show name and email info
create users.html insdie templates folder
![image](/blog/assets/images/2021102402.png)


add /users route in app.py
```python
(...)
@app.route('/users')
def users():
    # Fetch data from database
    cur = mysql.connection.cursor()
    resultValue = cur.execute("SELECT * FROM users")
    if resultValue > 0:
        userDetails = cur.fetchall()
        return render_template('users.html', userDetails=userDetails)

    return 'No data available'

(...)
```

`docker-compose up` and go to localhost:5001/users, check if you see the users info

#### 14. finally, in order to render user page after user submitted thier info
import redirect and change the following code inside app.py
```
from flask import Flask, render_template, request, redirect # ADD REDIRECT
(...)

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method =='POST':
        (...)
        return redirect('/users')   # CHANGE HERE!
    return render_template('index.html')

```

(Reference:
https://stavshamir.github.io/python/dockerizing-a-flask-mysql-app-with-docker-compose/
https://www.youtube.com/watch?v=6L3HNyXEais)


##### Project Code
[https://github.com/risatoy/demo-docker-flask-mysql](https://github.com/risatoy/demo-docker-flask-mysql)