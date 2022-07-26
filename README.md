# Dockerize Django and Postgres

## Introduction

Dockerize Django and Postgres is a template of how Django and Postgres may be run under Docker.

The purpose of the template is to quickly be able to develop a Django project using a Postgres database.

Using the approach in this project, neither Python, pip, Postgres, nor Django is necessary to install locally on the computer. Everything is running inside of Docker. All that is necessary is Docker. The `docker-compose.yml` file will use `volumes` to create a copies of the Django project and the Postgres database in your local directory, where your `docker-compose.yml` file is stored. Any changes you do in those directories will be reflected in the Django app that is running in a Docker container.

## Requirements

Docker.

## How to use

### Create a new Django project:

If you want to create a new project, run the command (do not forget the dot at the end):

```
sudo docker-compose run web django-admin startproject <project_name> .
```

If you are running Docker on Linux, the files django-admin creates are owned by the `root` user. This happens because the container runs as the `root` user. Change the ownership of the new files created by docker in your local directory with the command:

```
sudo chown -R $USER:$USER <project_name> manage.py
```

Do not change the permission of the `data` folder where Postgres has its file, otherwise Postgres will not be able to start due to permission issues.

### Connecting to a Postgres database

In your project directory, edit the `<project_name>/settings.py` file.

Replace the DATABASES = ... with the following:

```
import os

[...]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('POSTGRES_NAME'),
        'USER': os.environ.get('POSTGRES_USER'),
        'PASSWORD': os.environ.get('POSTGRES_PASSWORD'),
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

These settings are determined by the postgres Docker image specified in docker-compose.yml.

### Run a test with the Django development server

You should now be able to run the default Django app on URL `http://localhost:8000` with the command:

```
docker-compose up
```

### Shutting down the Django development server

Either run

```
docker-compose down
```

in a new terminal window, under the directory where you have your `docker-compose.yml` file.

ALternatively, press `Ctrl+C`.

### Create a new app

To create a new app, run:

```
sudo docker-compose run web python manage.py startapp <app_name>
```

As in the case of creating a project, Docker on Linux by default creates files and folders owned by the `root` user. Change the ownership of the newly created app directory with:

```
sudo chown -R $USER:$USER <app_name>
```

### Migration

Make migrations using docker-compose:

```
docker-compose run web python manage.py makemigrations
```

Migrate using docker-compose:

```
docker-compose run web python manage.py migrate
```

Alternatively, you can also make migrations from within the docker container. First, you need to make sure the containers are running with `docker-compose up`, then you need to get inside the container, for instance by running:

```
docker exec -it <container> bash
```

Then, you can simply run all the migration commands natively in the container's bash terminal.

```
python manage.py makemigrations
python manage.py migrate
```

## Acknowledgements

The project is an adaption of the official Docker documentation, which you can read [here](https://docs.docker.com/samples/django/)
