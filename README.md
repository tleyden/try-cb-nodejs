try-cb-nodejs
===============

A sample application and dataset for getting started with Couchbase 4.0.  The application runs a single page UI for demonstrating query capabilities.   The application uses Couchbase Server +  Node.js + Express + Angular and boostrap.   The application is a flight planner that allows the user to search for and select a flight route (including return flight) based on airports and dates. Airport selection is done dynamically using an angular typeahead bound to cb server query.   Date selection uses date time pickers and then searches for applicable air flight routes from a previously populated database.  

## Installation and Configuration
The steps below assume you are running a standalone couchbase instance running kv, indexing, and query services on the same server where the node application will also be running.  The config.json file in the root of this application can be edited to handle more complex topologies such as running couchbase server inside a vm.   
 - [1] Install a Couchbase Server, with integrated query service, and start the server.   There is no need to manually configure the server through the admin UI, steps 3 and 4 (below) will **automatically** provision couchbase and the application.
 - [2] Install Node.js
 - [3] Make a directory, clone this repo, install dependencies, start the application.  From a terminal:  
        mkidr ~/try-cb   
        git clone https://github.com/ToddGreenstein/try-cb-nodejs.git ~/try-cb   
        cd ~/try-cb   
        npm install   
        node app.js   
 - [4] Open a new terminal and run: curl -v -X POST http://127.0.0.1:3000/api/status/provisionCB
 - [5] Open a browser and load the url http://localhost:3000

## REST API DOCUMENTATION
#### GET /api/airport/findAll?search=<_search string_> [**RETURNS: {"airportname":"<_airport name_>"} for typeahead airports passed in the query string in the parameter "search"**] 	
--Used for Typeahead							
--Queries for Airport by Name, FAA code or ICAO code.

#### GET /api/flightPath/findAll?from=<_from airport_>&to=<_to airport_>&leave=<_leave date_>&ret=<_return date_> [**RETURNS: {"sourceairport":"<_faa code_>","destinationairport":"<_faa code_>","name":"<_airline name_>","equipment":"<_list of planes airline uses for this route_>"} of available flight routes**]
--Populates the available flights panel on successful query.  
--Queries for available flight route by FAA codes, and joins against airline information to provide airline name.  

#### POST /api/status/provisionCB [**RETURNS: {JSON OBJECT} indicating complete**]
--Loads the dataset dynamically based on options in the "config.json" file.   
--If dataset="repo", the application will build a sample bucket called "travel-sample" by loading raw json formatted air travel documents from the file in try-cb/model/raw/rawJsonAir.js and dynamically build scheduling information.  This is useful for learning how to programitically build a bucket, perform ingestions of data and how to become familiar with the CB SDK API.  
--If dataset="embedded", the application load the above information from the included sample bucket within couchbase known as "travel-sample"

## Running under Docker

### Create volume dir

```
$ rm -rf /tmp/data && mkdir /tmp/data
```

### Start Couchbase Server

```
$ docker run --name couchbase -v /tmp/data:/opt/couchbase/var -p 8091:8091 -p 8092:8092 -p 8093:8093 -d couchbase/server:enterprise-4.0.0-dp
```

To view Couchbase logs, run:

```
$ tail -f /tmp/data/lib/couchbase/logs/*
```

### Initialize Couchbase Server

```
$ docker run -ti --rm --entrypoint="/bin/bash" --link couchbase:couchbase couchbase/server:enterprise-4.0.0-dp
```

From inside the docker container run:

```
[root@96e4990f51c2 /]$ curl https://gist.githubusercontent.com/tleyden/059fc4e044bf0fae59cb/raw/1ca045135d7ca74ca201324634fa71f9bc37e996/gistfile1.sh | bash
```

Exit from the docker container

```
$ exit
```

### Start Travel App

Start travel app.

```
$ docker run -d --link couchbase:couchbase -p 3000:3000 tleyden5iwx/try-cb-nodejs 
```

### Initialize Travel App database

Run this on the host where the container was kicked off from.

```
$ curl -X "POST" "http://localhost:3000/api/status/provisionCB"
```

If you run `docker logs -f $container_id` and pass the container id of the `tleyden5iwx/try-cb-nodejs` container, you should see logs similar to [these](https://gist.github.com/tleyden/dbafd8a84176d52f0d0b)

### View Travel App website

In your browser, go to http://localhost:3000 and you should see the Travel App website.

## Running with docker-compose

If you use boot2docker
```
$ boot2docker ssh
```

```
$ rm -rf /home/docker/couch && mkdir /home/docker/data
```

### Start app

```
$ docker-compose up
```

### Initialize Couchbase Server

```
$ docker run -ti --rm --entrypoint="/bin/bash" --link trycbnodejs_couchbase_1:couchbase couchbase/server:enterprise-4.0.0-dp
```

From inside the docker container run:

```
[root@96e4990f51c2 /]$ curl https://gist.githubusercontent.com/tleyden/059fc4e044bf0fae59cb/raw/1ca045135d7ca74ca201324634fa71f9bc37e996/gistfile1.sh | bash
```

Exit from the docker container

```
$ exit
```

### Initialize Travel App database

Run this on the host where the container was kicked off from.

```
$ IP=`boot2docker ip`
$ curl -X "POST" "http://$IP/api/status/provisionCB"
```

### View Travel App website

In your browser, go to http://<boot2docker ip> and you should see the Travel App website.
