# Docker Training - Lab 3 - Creating a Container

This lab will walk through the process to setup a Dockerfile and build a container from scratch.

## Prerequisites

For this lab and all remaining labs, you will need a context directory to work from for your Dockerfile builds.  Each Dockerfile and its context needs a seperate folder. I would recommend on Windows something like: `c:\docker-labs` for the root folder with a sub folder for each context under that.

The remained of this lab will assume this root folder for your labs.

## 1 - Create a build context folder

Create a folder called `c:\docker-labs\lab-3-httpd`.  In this folder, create a file called `Dockerfile`, with *no file extension*.  

The contents of `Dockerfile` should be:

```Dockerfile
FROM httpd
```

This Dockerfile really isn't going much.  It is telling the Docker build agent to create a new container based off of httpd.  However, it isn't actually executing any aditional steps.

To see what this build does, let's run the following command from `c:\docker-labs\lab-3-httpd`:

```
docker build -t lab-3-httpd:empty .
```

The first statement builds your docker container.  The `-t lab-3-httpd:empty` labels this as `lab-3-http` with the version tag of `empty`.  (Note that if you remove the version tag, Docker assumes `:latest`) 

The `.` parameter supplies the build context to use, in this case the local folder.  You could also run `docker build -t lab-3-httpd c:\docker-labs\lab-3-https` to achieve the same effect.

Try running this command a couple of times.  You should note that it continues to build a container with the same digest number.  That is because Docker, by default, does not rebuild image layers if rhere is a locally cached version that has the same build context.

To test this container, let's run the following command.

```
docker run -p 10801:80 -d --rm --name lab-3-httpd lab-3-httpd:empty
```

Now, browse to the following URL to confirm that this application works:

http://localhost:10801

Having confirmed that this is working, let's stop this named container.

## 2 - Adding a new file to httpd

As exciting as it is to see **It works!**, we probably want our container to do a little more. 

To this, let's first create a folder called `public-html` with a file called `index.html` with the following contents:

```html
<html>
  <head>
    <title>Awesome Docker Site</title>
  </head>
  <body>
    <em>Hello, Docker labs!</em>
  <body>
<html>
```
The name `public-html` isn't special. The only thing is it is a best practice to *not* put any files in the same folder as Dockerfile but instead include them in sub-folders.

Now, edit your Dockerfile as follows:

```Dockerfile
FROM httpd
COPY ./public-html/ /usr/local/apache2/htdocs/
```

Now, let's build and run this new container with the same image name but a new version tag:

```
docker build -t lab-3-httpd:hi-docker .
docker run -p 10801:80 -d --rm --name lab-3-httpd lab-3-httpd:hi-docker
```

Hit your test web address to confirm that the new page is working.  (You may need to do a `CTRL+F5` refresh if the page is cached).  Once you have tested, stop your container.

This step of the lab worked by specifying a base image, in this case `httpd`, and then copying files into that image via the `COPY` command.  We are still using the standard `httpd` entrypoint that starts a container with an entrypoint of the Apache httpd server.  We just overwrote the index.html file in `/use/local/apache2/htdocs/`.

## 3 - Custom command

Up until this point, we have used and modified existing base images without changing the entrypoint of the container.  Let's look now at how to set up a command.

Create a folder called `c:\docker-labs\lab-3-bash` and create the following files:

`Dockerfile`
```Dockerfile
FROM ubuntu
COPY ./scripts/my.sh /usr/scripts/my.sh
```

`scripts/my.sh` (Note, this file should be saved in Linux stil line endings.  In Visual Studio Code, you can select the `CRLF` item in the bottom right task bar and change to `LF` )
```bash
#!/bin/bash
echo Hello Docker Labs!
```

Build this container with the following command:

```
docker build -t lab-3-bash:nocmd .
```

Now, let's run this container by passing in the path to our script as a command parameter:

```
docker run --rm lab-3-bash:nocmd /usr/scripts/my.sh
```

This passes the script to the defined entrypoint of container. For `ubunutu`, this is the `bash` executable.

Since we want our container to always run this command when starting, let's edit our Dockerfile:

`Dockerfile`
```Dockerfile
FROM ubuntu
COPY ./scripts/my.sh /usr/scripts/my.sh
CMD "/usr/scripts/my.sh"
```

Now, we can build and run this container.  

```
docker build -t lab-3-bash:cmd .
docker run --rm lab-3-bash:cmd
```

We see that the same results happen.  This is because we have passed our command by default into the `bash` entrypoint.

## 4 - Custom entrypoint

Now, what happens if we pass a different command into our container we just created?

```
docker run --rm lab-3-bash:cmd ls
```

We see that our **Hello, Docker Labs!** output is no longer showing.  That is because the command line passed into the `docker run` command overrides the `CMD` directive in our container.  In some cases, such as with the `ubuntu` base image passing commands to `bash`, this is to be desired.  

However, we may want our container to *always* run our script.  Also, we may want to pass additional command line args to our script.  In both of these cases, we should change the `ENTRYPOINT` of our container.

Let's modify our Dockerfile:
`Dockerfile`
```Dockerfile
FROM ubuntu
COPY ./scripts/my.sh /usr/scripts/my.sh
ENTRYPOINT "/usr/scripts/my.sh"
```

Now, let's build and run our container:

```
docker build -t lab-3-bash:entrypoint .
docker run --rm lab-3-bash:entrypoint
```

If we try passing in a command, it doesn't change any behavior:

```
docker run --rm lab-3-bash:entrypoint
```

Something to note that in both the `CMD` and `ENTRYPOINT` methods here immediately exited the the contaier.  This is because Docker containers immediately terminate when their entrypoint exits.

## 5 - Run installation commands as part of build

Up until this point, we have just copied files into a container. In this section of the lab, we will execute "install" steps.

Create a folder called `c:\lab-3-python`.  Don't worry - you don't need to know Python for this lab.

Create the following files:

```Dockerfile```
```Dockerfile
FROM python:3.6
COPY ./py/ /py/
ENTRYPOINT [ "python3", "/py/demo.py"]
```

```py/demo.py```
```python
print('Hello, everybody!')
```

Now, run these commands to build and test our container:

```
docker build -t lab-3-python:hello .
docker run --rm lab-3-python:hello
```

As exciting as this is, let's do something better.  Let's use Python to build an API using the Flask framework.  To do this, replace your `demo.py` file as follows:

```python
from flask import Flask, request
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

class Sample(Resource):

    def get(self):
        return "hi"

api.add_resource(Sample, '/sample') 

if __name__ == '__main__':
     app.run(port=80, host='0.0.0.0')
```

Create and run this container...
```
docker build -t lab-3-python:flask .
docker run -p 10802:80 --rm --name lab-3-python lab-3-python:flask .
```

You see we get the following error: 

`ModuleNotFoundError: No module named 'flask'`

This is because the Python base image doesn't have the flask dependencies installed.

So, let's edit our Dockerfile to install them using a `RUN` command

```Dockerfile
FROM python:3.6
COPY ./py/ /py/
RUN pip install --upgrade pip
RUN pip install flask
RUN pip install flask_restful
ENTRYPOINT [ "python3", "/py/demo.py"]
```

Create and run the container...
```
docker build -t lab-3-python:flask-working .
docker run -p 10802:80 --rm --name lab-3-python lab-3-python:flask-working .
```

Now, browse this site and see that your Python container is executing your request:

http://localhost:10802/sample

## Lab Complete

In this lab, we have learned how to:

* Create a Dockerfile
* Build a tagged image from a Dockerfile
* Copy files into an image from a build context
* Override a base image command via the `CMD` command
* Override a base image entry point via the `ENTRYPOINT` command
* Execute steps as part of a build using the `RUN` command