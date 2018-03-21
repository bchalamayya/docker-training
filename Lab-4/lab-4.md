# Docker Training - Lab 4 - Docker Compose

In this lab, we will look at using the Docker Compose package to manage creating a pre-defined stack of Docker containers that make up an application stack.

## 1 - Simple Docker Compose

Create a folder called `c:\docker-labs\lab-4-compose-httpd`. In it, place the following file:

```docker-compose.yml```
```yaml
version: "2"

services:
  httpd:
    image: httpd
    ports:
      - 10804:80
```

Now, run the following command from that folder:

```
docker-compose up -d
```

This will start the services up in a detached process.

Now, try browsing to:

http://localhost:10804

You can see the logs of your containers by running the following command:

```
docker-compose logs
```

And finally, to shut your containers down, you can use this command:

```
docker-compose down
```

## 2 - Docker Compose Build

Create a folder called `c:\docker-labs\lab-4-compose-build`. In it, place the following files:

```docker-compose.yml```
```yaml
version: "2"

services:
  httpd:
    build:
      context: ./awesome-httpd/.
    ports:
      - 10805:80
```

```awesome-httpd/Dockerfile```
```Dockerfile
FROM httpd
COPY ./public-html/ /usr/local/apache2/htdocs/
```

```awesome-httpd/public-html/index.html```
```html
<html>
  <head>
    <title>Awesome Docker Site</title>
  </head>
  <body>
    <em>Hello again, Docker labs!</em>
  <body>
<html>
```

Now, run the following command from that folder:

```
docker-compose up -d
```

Next, browse to:

http://localhost:10805

Next, edit the `index.html` file -- put something sarcastic in there, and run the following command.

```
docker-compose up -d
```

Notice how you *don't* have to stop the containers -- Docker Compose is looking to see if anything changed.  It tell you that everything is up to date.  Well, you changed a file?  What gives?

Docker Compose is not re-building the container from the build context.  By default, Docker Compose will only build a container if it does not yet exist.  So, let's run the following command:

```
docker-compose up -d --build
```

You should see when browsing to your site that your new sarcastic statements are in place.

## 3 - Multi-stack compose

Where Compose really shines is with multi-component stacks.  For our final step of this lab, let's get a stack up and running with both httpd and Nginx

Create a folder called `c:\docker-labs\lab-4-compose-stack`. In it, place the following file:

```docker-compose.yml```
```yaml
version: "2"

services:
  httpd:
    image: httpd
    ports:
      - 10806:80

  nginx:
    image: nginx
    ports:
      - 10807:80
```

Now, run the following command from that folder:

```
docker-compose up -d
```

This will start the services up in a detached process.

Now, try browsing to:

http://localhost:10806
http://localhost:10807

To wrap this lab up, go run `docker-compose down` for all of your environments.

## Lab Complete

In this lab, we have learned how to:

* Create a Docker Compose yml file
* Run a compose definition
* View logs of a compose stack
* Use Compose to build a container image
* Use Compose to create multiple containers as one stack
