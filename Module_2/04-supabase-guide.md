# Databases and APIs: Definition and CRUD

A **database** is a system where application data is stored and organized. An **API (Application Programming Interface)** acts as a bridge between the application and the database, allowing software to request and manipulate data securely. Specifically, a *database API* provides a standardized interface to communicate with the underlying data engine. Through these APIs we can perform the basic **CRUD** operations –Create, Read, Update and Delete records– transparently. For example, the API can expose methods like `InsertRecord`, `UpdateRecord`, `DeleteRecord`, etc., or in Django ORM terms methods like `.create()`, `.get()`, `.save()`, `.delete()`. These CRUD operations allow the application to manage data stored in the database. In essence, APIs facilitate data flow between the database and application code, isolating low-level details (connection, SQL queries, error handling) from the developer. This makes development more agile, secure, and portable, as the application can change databases without modifying all internal logic (thanks to the abstraction layer provided by the API).

## What is Supabase?

Supabase is an **open-source backend platform** that facilitates the creation of modern applications. In essence, it provides a set of services (database, authentication, storage, etc.) similar to Firebase, but built on PostgreSQL. It was designed as an **open-source alternative to Firebase**: it uses a PostgreSQL relational database, so it supports complex queries, text searches, transactions, etc. For example, Kinsta notes that *"Supabase describes itself as an 'open-source alternative to Firebase'. It's based on a relational database, using PostgreSQL for its functionality and scalability"*. Among its main features are real-time database (change subscriptions), complete authentication (email, OAuth, JWT), file storage, and very importantly, **automatic generation of RESTful APIs** based on defined tables. When creating tables in Supabase, it automatically exposes REST endpoints (and optionally GraphQL) to make CRUD queries without configuring anything extra. In summary, Supabase offers a ready-to-use backend, with managed PostgreSQL in the cloud and APIs ready to interact with the database, which greatly accelerates application development.

## Creating an account and project in Supabase

To use Supabase, you must first **register** on their website. Luis Llamas' guide indicates the basic steps: 1) Register/sign in at [supabase.com](https://supabase.com) and confirm your email. 2) Click **"New Project"**, assign a name and password for the database. 3) Once the project is created, Supabase will show you the credentials (username, password) and the API endpoint URL for your database. It's important to copy the URL (for example, `https://xyzcompany.supabase.co`) and the **API key** that Supabase provides; these will be used later to connect to the service. Supabase offers a generous free plan: you can create 2 projects, each with up to 500MB of database and 1GB of file storage. This is ideal for starting without needing your own infrastructure.

## Connecting Supabase with Django

Supabase exposes your PostgreSQL database externally, so Django can connect **directly** as with any other Postgres DB. Suppose we want to use Supabase as the main database of our Django app. First, we install the PostgreSQL adapter for Python:

```bash
pip install psycopg2-binary
```

This allows Django to communicate with PostgreSQL. Then, in the Supabase panel (Project Settings → Database → Connection), we get the connection string. It will be something like:

```
postgres://postgres:YOUR_PASSWORD@<host>.supabase.co:5432/postgres
```

With this data, we edit Django's `settings.py`, for example:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',  # Use PostgreSQL
        'NAME': 'postgres',            # Database name (default 'postgres')
        'USER': 'postgres',            # Database user
        'PASSWORD': 'YOUR_PASSWORD',   # Password assigned in Supabase
        'HOST': 'abcdef.supabase.co',  # Supabase instance host
        'PORT': '5432',
    }
}
```

In this example, replace the values with those from your project. Once configured, Django will use Supabase as the backend: when running `manage.py migrate`, it will create tables in the remote database; the application will read/write data in Supabase as if it were any local Postgres. This option is very simple because it takes full advantage of Django's ORM without additional changes.

### Option 2: Using Supabase's RESTful/GraphQL API

Supabase also offers ready-to-use APIs: it automatically exposes your tables through a REST API (via PostgREST) and a GraphQL API. This means you **don't need to use Django's ORM**: you can make HTTP calls from your Python code to manipulate data. For example, you could use the [supabase-py](https://github.com/supabase-community/supabase-py) library (client in development) that internally uses the REST API, or simply use the `requests` module. In the Supabase panel, you'll see that each table has a REST endpoint at `https://<PROJECT>.supabase.co/rest/v1/`. A simple example is creating a `supabase_utils.py` file in your Django app with functions to call the API:

```python
import requests
from django.conf import settings

SUPABASE_URL = settings.SUPABASE_URL    # e.g. 'https://abc.supabase.co'
SUPABASE_KEY = settings.SUPABASE_KEY    # public key (anon key)

headers = {
    "apikey": SUPABASE_KEY,
    "Content-Type": "application/json",
    "Authorization": f"Bearer {SUPABASE_KEY}"
}

def fetch_from_supabase(table):
    url = f"{SUPABASE_URL}/rest/v1/{table}"
    response = requests.get(url, headers=headers)
    return response.json()

def insert_to_supabase(table, data):
    url = f"{SUPABASE_URL}/rest/v1/{table}"
    response = requests.post(url, headers=headers, json=data)
    return response.json()
```

With this utility, you can, for example, in a Django view call `fetch_from_supabase('my_table')` to get JSON data from the table, or `insert_to_supabase('my_table', {'field': 'value'})` to insert records. This technique doesn't use Django models but direct API calls, which avoids configuring local migrations. Each request must include the API key in the headers (as shown in the previous code).

## Defining models in Django with Supabase

If you choose to use the direct connection to Supabase's database (option 1), you define your **Django models normally**. For example:

```python
from django.db import models

class Task(models.Model):
    name = models.CharField(max_length=100)
    completed = models.BooleanField(default=False)
```

Then run `python manage.py makemigrations` and `migrate`: Django will create the corresponding table in Supabase. If, instead, your Supabase database already has existing tables and you don't want to recreate them, you can use the command `python manage.py inspectdb` to automatically generate models from the current schema. This creates `models.Model` classes (with `Meta.managed = False` to indicate that Django won't manage them) based on existing tables. In summary, working with Django models is identical to using any PostgreSQL: you define classes with their fields, and Django handles CRUD operations using the ORM.

## CRUD operations from Django using Supabase

Once the connection is configured, **CRUD operations are done the same as in normal Django**. With the ORM, you can create, read, update, and delete model objects. For example:

```python
# Create a record
t = Task.objects.create(name='Buy bread', completed=False)

# Query (Read)
all_tasks = Task.objects.filter(completed=False)

# Update
t = Task.objects.get(pk=1)
t.completed = True
t.save()

# Delete
t.delete()
```

These methods internally execute SQL statements on the Supabase database. As Django docs indicate, *"Django automatically gives you a database abstraction API that lets you create, retrieve, update and delete objects"*. Equivalently, if you're using the REST API option, you would perform CRUD operations by calling your utility functions. For example, a Django *view* can call `fetch_from_supabase('task')` to get data and return it in JSON (doing the serialization yourself).

## Security, API keys, and authentication

When using Supabase, you must carefully handle **keys** and permissions. Supabase provides at least two main keys: the "anon" key (public/publishable) and the `service_role` key (privileged). The "anon" key can be included in clients and is used for ordinary calls; it's associated with the `anon` role in PostgreSQL, which applies the RLS (Row Level Security) policies you define in each table. It's fundamental to activate **RLS on all tables** and define policies that allow access only as appropriate. For example, you can create a "public read-only" policy for anonymous clients.

On the other hand, the `service_role` key provides **full access** (bypassing any RLS), so it **should never be exposed publicly** or included in client code. This key should only be used in secure parts of your system (for example, server functions or internal tasks). Always store sensitive keys in environment variables (or secret services) and **don't commit them to the repository**. In general, Supabase recommends treating keys as secrets: sharing them only through secure channels and rotating them if necessary.

Regarding **user authentication**, Supabase offers its own system (email users, OAuth, etc.) integrated with JWT. If you want to use Supabase's authentication from Django, you'll need to verify the JWT tokens that Supabase returns. One strategy is to write a custom *middleware* in Django that reads the `Authorization` header, validates the JWT with Supabase's public key, and assigns `request.user` accordingly. This way you take advantage of Supabase's registration/login flow within your Django application. However, this can be complex, as Django has its own authentication system (with sessions, permissions, etc.). Often it's simpler to use Django's native auth system and, if necessary, sync users with Supabase (for example, using triggers to replicate them in the database).

