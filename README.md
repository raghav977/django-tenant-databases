# django-tenant-databases
Dynamic multi-tenant database management in Django with automatic database creation and migrations

## Introduction- What is multi tenant architecture?
Let's go from an example: Suppose you are making a SAAS application, for school management system through django, then what will be your first approach? You just make a django-starter project. Define models Like: Student,Teacher,Staffs and so on. Suppose a school named A buys this software from you, and started to store its data like student records then it will be stored in that student table under a same database. Again, Suppose a school Named B buys the same software from you, then the details of School B students will again be stored on the same table Student under a same database. What if your database gets corrupted? Your all data will be lost. Your single table has a lot of values, perhaps might work so slow then expected. To solve this problem, the engineers/programmers came with the idea of multi tenant

## Multi-database architecture
First of all, let's understand what is tenant? In a simple word tenant is just a fancy word for the single organization/customer. School A is one tenant, School B is another tenant. So, Now our approach will be making the Single database for each tenant. School A Will have its own dedicated database and School B as well.


## what will be we doing?
the user will prompt school name: Rising School
then the Rising school will be the database and all the models like Student, Staff, Teacher should auto migrate.
Similarly for other schools as well.

## How to acheive it?
## Activating the virtual environment
1. First of all, let's activate the virtual environment.
2. python -m venv env, this will create the env folder.
3. env\Scripts\activate
4. paste it to activate virtual environment, the verification of virtual environment is shown as the (env) in the beginning like below
5. <img width="780" height="98" alt="image" src="https://github.com/user-attachments/assets/f567d2db-3cf4-44cc-a35c-d9978030c015" />

## Installing django and creating django-starter-project
1. pip install django --> command to install the django
2. pip install mysqlclient
3. after the completion of django installation, let's make the django-project
4. django-admin startproject learningtenet --> learningtenet is the project name
5. After that the learningtenet project will be created, now to perform other commands, you have to be in the same level as the manage.py file is located. manage.py is always located inside your project folder navigate to your project folder,
6. cd learningtenet
7. <img width="1304" height="650" alt="image" src="https://github.com/user-attachments/assets/389e3c80-2938-40bb-8402-b3a1e278bfdc" />
8. Let's create an app called core
9. python manage.py startapp core
10. make sure to include it in settings.py installed_apps
11. <img width="1311" height="685" alt="image" src="https://github.com/user-attachments/assets/9e2a5bde-5143-4ea1-8a08-135daf2a9274" />

## Let's change the database from sqlite to mysql
By default, Django uses SQLite as its database. For this tutorial, we will use MySQL instead.

To use MySQL, first make sure it is installed on your system.
Download MySQL Community Server from the official site:
MySQL Downloads

After downloading, install and set up the MySQL Community Server properly.
Important: Don’t forget the password you set during installation — you will need it later for Django configuration.
make sure you create one database:
**create database tutorialtenet;**
Note: The latest versions of Django require a compatible version of MariaDB/MySQL. Using bundled packages like XAMPP may cause version conflicts or errors.
Comment out the existing sqlite configuration
<img width="952" height="366" alt="image" src="https://github.com/user-attachments/assets/19a2bfbe-575e-41e9-a7ce-0f441d501a27" />
and paste the code
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'tutorialtenet',     # database name here (make sure the database is already created)
        'USER': 'root',
        'PASSWORD': 'Root123@', #password you set during the setup
        'HOST': 'localhost',
        'PORT': '3306',
        
    }
}
Let's create a models : School, Student, Staff
Here, School is the tenant. The database will be created according to this tenant.

## creating the tenant model
let's create the model in models.py inside app core, for now will be creating the database according to school_name, for real life project make sure when the school is created generate the unique key and add that key to that school. so the school with same name can also be created.
from django.db import models

# Create your models here.
class Tenant(models.Model):
    school_name = models.CharField(max_length=200)
    
<img width="858" height="265" alt="image" src="https://github.com/user-attachments/assets/e09fab95-935f-4b88-b2b7-c6dba0abcfff" />

class Student(models.Model):
    name = models.CharField(max_length=200)

MAKE SURE TO PERFORM MAKEMIGRATIONS, MIGRATE
## creating the view:
when the user enters: "school1" and sends post request
we should take that school1 and make a db, and then migrate the model in that db.
Lets do it
def create_db(request):
    if request.method=='POST':
        school_name = request.POST.get('school_name')
        actual_db_query(school_name)

here in school_name we got school1
NOTE: in django all the database related things can be acheived through settings.DATABASES variable
now let's point that variable
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'tutorialtenet',     
        'USER': 'root',
        'PASSWORD': 'Root@123',
        'HOST': 'localhost',
        'PORT': '3306',
        
    }
}
Look at the NAME key the value inside the NAME key is our database name, Now if from any where we simply just insert our school_name inside that NAME value then we can easily make the tables inside that database
But the thing is only DATABASES['default']['NAME']='school_name' -->  wont work because to register the db inside the NAME the db should be already exist, it means we have to already use the query 
## create database {school_name};
so to use the query we need the connection object from django. that is found in from django.db package let's import connections and transaction (i will discuss on below why transaction?)
One thing: The query can only be executed through the help of already created database, and here tutorialtenet, is already created database so, with the help of that DATABASES['deafult']['NAME'] we will execute our create database query 
BUT THE THING IS WE CAN ONLY QUERY WITH THE help of connections object.
from django.db import connections, transaction
def actual_db_query(school_name):
  try:
      with connections['default'].cursor() as cursor:
        cursor.execute(f"CREATE DATABASE `{school_name}`")
        transaction.commit()
        // with this code our database finally created here connections is the instance of that databases= variable. To perform the query we must need the cursor() function so we used it to execute the      //query.transaction.commit() makes the commit final and finalized the database.


Now we need to migrate al the models inside this database for this what we need is
we need to call the function named call_command(), inside its parameter you write the command, and the name of database
call_command('migrate', database=school_name)
but django only understand that database name that is registered inside the NAME: of that databases variable so we need to make a copy of it.


DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'tutorialtenet',     
        'USER': 'root',
        'PASSWORD': 'Root@123',
        'HOST': 'localhost',
        'PORT': '3306',
        
    }
}
from this now let's take the default key's value

from django.conf import settings
default_values = settings.DATABASES['default']
now to not reflect the change to actual default values we need to shallow copy it
new_db_settings = default_values.copy()
now inside of the NAME lets replace our own school_name
that is
new_db_settings['NAME'] = school_name
connections.databases[school_name] = new_db_settings

Before running call_command('migrate', ...), Django needs to know about this new DB config in memory AS IT IS JUST A SHALLOW COPY:

now the database name is successfully inside the NAME 
so the call_command will work
call_command('migrate', database=school_name)
so, the final code will be:

from django.shortcuts import HttpResponse
from django.conf import settings
from django.db import connections, transaction
from django.core.management import call_command
from django.views.decorators.csrf import csrf_exempt
import json
@csrf_exempt
def create_db(request):
    if request.method == 'POST':
        body = json.loads(request.body)
        school_name = body.get('school_name')
        print("This is school_name", school_name)

        if not school_name:
            return HttpResponse("School name is required", status=400)

        try:
            actual_db_query(school_name)
            return HttpResponse("Database created successfully!")
        except Exception as e:
            return HttpResponse(f"Database creation failed: {str(e)}", status=500)

def actual_db_query(school_name):
    with connections['default'].cursor() as cursor:
        cursor.execute(f"CREATE DATABASE `{school_name}`")
        transaction.commit()

    # Copy default DB config and update NAME
    default_db = settings.DATABASES['default']
    new_db_settings = default_db.copy()
    new_db_settings['NAME'] = school_name

    # Register the new DB settings in Django
    connections.databases[school_name] = new_db_settings

    # Run migrations on the new database
    call_command('migrate', database=school_name)


LET'S CHECK
<img width="892" height="472" alt="image" src="https://github.com/user-attachments/assets/3140b94e-8424-4177-b25a-3fbf4813e9b0" />
<img width="493" height="542" alt="image" src="https://github.com/user-attachments/assets/b6c7ac51-240d-4654-82f5-b47de5c26c21" />

<img width="544" height="443" alt="image" src="https://github.com/user-attachments/assets/21be3b64-5795-4dcd-a4d1-4e14757553ce" />

Let's try another school_name="teresa"
<img width="894" height="600" alt="image" src="https://github.com/user-attachments/assets/c8ec9731-1f0a-4009-ac0c-8d3837451d22" />
<img width="487" height="560" alt="image" src="https://github.com/user-attachments/assets/ecee9d65-9980-430e-a08b-d12a7fccb7a2" />
<img width="533" height="430" alt="image" src="https://github.com/user-attachments/assets/ece0fcd9-c6a9-473e-a12a-e313499ca21e" />


THANK YOU.






