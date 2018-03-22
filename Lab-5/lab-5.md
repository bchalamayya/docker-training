# Docker Training - Lab 5 - Docker Data Volumes

In this lab, we will look at how state management works for Docker

## 1 - Creating a Stateful Container Stack

Create a folder called `c:\docker-labs\lab-5-volumes`. In it, place the following file:

```docker-compose.yml```
```yaml
version: "2"

services:
  neo4j:
    image: neo4j
    ports:
      - 7474:7474
      - 7687:7687
```

Start the container up with:

`docker-compose up -d`

Now this startup will take a little longer.

Once the container has started, open the following link:

http://localhost:7474

Your username is `neo4j`, with `neo4j` as your password.

Log in and change your password.

Now, click on the `Write Code` button.  In the `Movie Graph` sample, click `Create a graph`.  Play around for a bit.  

When you are done, close your browser window and stop your container:

`docker-compose down`

And then restart it:

`docker-compose up -d`

Re-open the browser.  Note that we are back to our initial start state.

#2 - Enable Data Volume

Modify your compose file as follows:

```docker-compose.yml```
```yaml
version: "2"

services:
  neo4j:
    image: neo4j
    ports:
      - 7474:7474
      - 7687:7687
    volumes:
      - neo4jdata:/var/lib/neo4j/data

volumes:
  neo4jdata:
```

Now stop and restart your container.  Walk through some stateful changes and stop/restart your container again.  You should see that statefulness is maintained between sessions.

#3 - CLI `volume` Command

To see where this volume is stored, run teh following command:

```
docker volume ls
```

This shows you what data volumes are stored on your machine.

## Lab Complete

In this lab, we have learned how to:

* Create a data volume in a compose file
* Use a data volume in a container
* View volumes on your host machine