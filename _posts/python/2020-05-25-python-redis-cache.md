---
title: Effortless API Request Caching with Python & Redis
toc: true
comments: true
layout: post
description: Caching MapBox API requests with Python & Redis
categories: [Python]
---

Recently, I was working with [MapBox](https://www.mapbox.com/)'s [Route Optimization API](https://docs.mapbox.com/api/navigation/#optimization). Basically, it tries to solve the [traveling salesman problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem) where you provide the API with coordinates of multiple places and it returns a duration-optimized route between those locations. This is a perfect usecase where [Redis](https://redis.io/) caching can come handy. Redis is an blazing fast, lightweight in-memory database with additional persistence options; making it a perfect candidate for the task at hand. Here, caching can save you from making redundant API requests and also, it can dramatically improve the response time as well.

I found that in my country, the optimized routes returned by the API do not change dramatically for at least for a couple of hours. So the workflow will look something like this:

* Caching the API response in Redis using the key-value data structure. Here the requested coordinate-string will be the key and the response will be the corresponding value
* Setting a timeout on the records
* Serving new requests from cache if the records exist
* Only send a new request to MapBox API if the response is not cached and then add that response to cache


## Setting Up Redis & RedisInsight

To proceed with the above workflow, you'll need to install and setup Redis database on your system. For monitoring the database, I'll be using  [RedisInsight](https://redislabs.com/redisinsight/). The easiest way to setup Redis and RedisInsight is through [docker](https://www.docker.com/) and [docker-compose](https://docs.docker.com/compose/). Here's a docker-compose that you can use to setup everything with a single command.

```yml
# docker-compose.yml

version: "3.2"
services:
  redis:
    container_name: redis-cont
    image: "redis:alpine"
    command: redis-server --requirepass ubuntu
    environment:
      - REDIS_PASSWORD=ubuntu
      - REDIS_REPLICATION_MODE=master
    ports:
      - "6379:6379"
    volumes:
      # save redisearch data to your current working directory
      - ./redis-data:/data
    command:
      # Save if 100 keys are added in every 10 seconds
      - "--save 10 100"
      # Set password
      - "--requirepass ubuntu"

  redisinsight: # redis db visualization dashboard
    container_name: redisinsight-cont
    image: redislabs/redisinsight
    ports:
      - 8001:8001
    volumes:
      - redisinsight:/db

volumes:
  redis-data:
  redisinsight:
```

The above `docker-compose` file has two services, `redis` and `redisinsight`. I've set up the database with a dummy password `ubuntu` and made it persistent using a folder named `redis-data` in the current working directory. The database listens in localhost's port `6379`. You can monitor the database using `redisinsight` in port 8000. To spin up Redis and RedisInsight containers, run:

```
docker-compose up -d
```

This command will start the database and monitor accordingly. You can go to this `localhost:8000` link using your browser and connect redisinsight to your database. After connecting your database, you should see a dashboard like this in your redisinsight panel:

![Screenshot from 2020-05-23 19-23-21](https://user-images.githubusercontent.com/30027932/82731781-f30b1b00-9d2a-11ea-8c72-62a4753bc5f9.png)


## Preparing Python Environment

For local development, you can set up your python environment and install the dependencies using pip. Here, I'm on a Linux machine and using virtual environment for isolation. The following commands will work if you're on a \*nix based system and have `python 3.8` installed on your system. This will install the necessary dependencies in a virtual environment:

```bash
python3.8 -m venv venv
source venv/bin/activate
pip install redis httpx
```

## Workflow

### Connecting Python Client to Redis

Assuming the database server is running and you've installed the dependencies, the following snippet connects `redis-py` client to the database.


```python
import redis
import sys

def redis_connect() -> redis.client.Redis:
    try:
        client = redis.Redis(
            host="localhost",
            port=6379,
            password="ubuntu",
            db=0,
            socket_timeout=5,
        )
        ping = client.ping()
        if ping is True:
            return client
    except redis.AuthenticationError:
        print("AuthenticationError")
        sys.exit(1)



client = redis_connect()
```

The above excerpt tries to connect to the `Redis` database server using the port `6379`. Notice, how I'm providing the password `ubuntu` via the `password` argument. Here, `client.ping()` helps you determine if a connection has been established successfully. It returns `True` if a successful connection can be established or raises specific errors in case of failures. The above function handles `AuthenticationError` and prints out an error message if the error occurs. If everything goes well, running the `redis_connect()` function will return an instance of the `redis.client.Redis` class. This instance will be used later to set and retrieve data to and from the redis database.

### Getting Route Data From MapBox API

The following function strikes the MapBox Route Optimization API and collects route data.

```python
import httpx


def get_routes_from_api(coordinates: str) -> dict:
    """Data from mapbox api."""

    with httpx.Client() as client:
        base_url = "https://api.mapbox.com/optimized-trips/v1/mapbox/driving"

        geometries = "geojson"
        access_token = "Your-MapBox-API-token"

        url = f"{base_url}/{coordinates}?geometries={geometries}&access_token={access_token}"

        response = client.get(url)
        return response.json()
```

The above code uses Python's [httpx](https://github.com/encode/httpx) library to make the get request. Httpx is almost a drop-in replacement for the ubiquitous [Requests](https://github.com/psf/requests) library but way faster and has async support. Here, I've used context manager `httpx.Client()` for better resource management while making the `get` request. You can read more about context managers and how to use them for hassle free resource management [here](https://rednafi.github.io/digressions/python/2020/03/26/python-contextmanager.html).

The `base_url` is the base url of the route optimization API and the you'll need to provide your own access token in the `access_token` field. Notice, how the `url` variable builds up the final request url. The `coordinates` are provided using the `lat0,lon0;lat1,lon1;lat2,lon2...` format. Rest of the function sends the http requests and converts the response into a native dictionary object using the `response.json()` method.


### Setting & Retrieving Data to & from Redis Database

The following two functions retrieves data from and sets data to redis database respectively.

```python
from datetime import timedelta


def get_routes_from_cache(key: str) -> str:
    """Get data from redis."""

    val = client.get(key)
    return val


def set_routes_to_cache(key: str, value: str) -> bool:
    """Set data to redis."""

    state = client.setex(key, timedelta(seconds=3600), value=value, )
    return state
```

Here, both the keys and the values are strings. In the second function, `set_routes_to_cache`, the `client.setex()` method sets a timeout of 1 hour on the key. After that the key and its associated value get deleted automatically.

### The Central Orchestration

The `route_optima` function is the primary agent that orchestrates and executes the caching and returning of responses against requests. It roughly follows the execution flow shown below.

![redis-cache](https://user-images.githubusercontent.com/30027932/82735908-1ba10e00-9d47-11ea-9e86-ac1fbc63628f.png)



When a new request arrives, the function first checks if the return-value exists in the Redis cache. If the value exists, it shows the cached value, otherwise, it sends a new request to the MapBox API, cache that value and then shows the result.

```python

def route_optima(coordinates: str) -> dict:

    # First it looks for the data in redis cache
    data = get_routes_from_cache(key=coordinates)

    # If cache is found then serves the data from cache
    if data is not None:
        data = json.loads(data)
        data["cache"] = True
        return data

    else:
        # If cache is not found then sends request to the MapBox API
        data = get_routes_from_api(coordinates)

        # This block sets saves the respose to redis and serves it directly
        if data["code"] == "Ok":
            data["cache"] = False
            data = json.dumps(data)
            state = set_routes_to_cache(key=coordinates, value=data)

            if state is True:
                return json.loads(data)
        return data
```


### Exposing As an API

This part of the code wraps the original Route Optimization API and exposes that as a new endpoint. I've used [fastAPI](https://fastapi.tiangolo.com/) to build the wrapper API. Doing this also hides the underlying details of authentication and the actual endpoint of the MapBox API.

```python
from fastapi import FastAPI


app = FastAPI()


@app.get("/route-optima/{coordinates}")
def view(coordinates):
    """This will wrap our original route optimization API and
    incorporate Redis Caching. You'll only expose this API to
    the end user. """

    # coordinates = "90.3866,23.7182;90.3742,23.7461"

    return route_optima(coordinates)
```

### Putting It All Together

```python
# app.py

import json
import sys
from datetime import timedelta

import httpx
import redis
from fastapi import FastAPI



def redis_connect() -> redis.client.Redis:
    try:
        client = redis.Redis(
            host="localhost",
            port=6379,
            password="ubuntu",
            db=0,
            socket_timeout=5,
        )
        ping = client.ping()
        if ping is True:
            return client
    except redis.AuthenticationError:
        print("AuthenticationError")
        sys.exit(1)


client = redis_connect()


def get_routes_from_api(coordinates: str) -> dict:
    """Data from mapbox api."""

    with httpx.Client() as client:
        base_url = "https://api.mapbox.com/optimized-trips/v1/mapbox/driving"

        geometries = "geojson"
        access_token = "Your-MapBox-API-token"

        url = f"{base_url}/{coordinates}?geometries={geometries}&access_token={access_token}"

        response = client.get(url)
        return response.json()


def get_routes_from_cache(key: str) -> str:
    """Data from redis."""

    val = client.get(key)
    return val


def set_routes_to_cache(key: str, value: str) -> bool:
    """Data to redis."""

    state = client.setex(key, timedelta(seconds=3600), value=value,)
    return state


def route_optima(coordinates: str) -> dict:

    # First it looks for the data in redis cache
    data = get_routes_from_cache(key=coordinates)

    # If cache is found then serves the data from cache
    if data is not None:
        data = json.loads(data)
        data["cache"] = True
        return data

    else:
        # If cache is not found then sends request to the MapBox API
        data = get_routes_from_api(coordinates)

        # This block sets saves the respose to redis and serves it directly
        if data["code"] == "Ok":
            data["cache"] = False
            data = json.dumps(data)
            state = set_routes_to_cache(key=coordinates, value=data)

            if state is True:
                return json.loads(data)
        return data


app = FastAPI()


@app.get("/route-optima/{coordinates}")
def view(coordinates: str) -> dict:
    """This will wrap our original route optimization API and
    incorporate Redis Caching. You'll only expose this API to
    the end user. """

    # coordinates = "90.3866,23.7182;90.3742,23.7461"

    return route_optima(coordinates)
```

You can copy the complete code to a file named `app.py` and run the app using the command below (assuming redis, redisinsight is running and you've installed the dependencies beforehand):

```
uvicorn app.app:app --host 0.0.0.0 --port 5000 --reload
```

This will run a local server where you can send new request with coordinates.

Go to your browser and hit the endpoint with a set of new coordinates. For example:

```
http://localhost:5000/route-optima/90.3866,23.7182;90.3742,23.7461
```

This should return a response with the coordinates of the optimized route.

```json
{
   "code":"Ok",
   "waypoints":[
      {
         "distance":26.041809241776583,
         "name":"",
         "location":[
            90.386855,
            23.718213
         ],
         "waypoint_index":0,
         "trips_index":0
      },
      {
         "distance":6.286653078791968,
         "name":"",
         "location":[
            90.374253,
            23.746129
         ],
         "waypoint_index":1,
         "trips_index":0
      }
   ],
   "trips":[
      {
         "geometry":{
            "coordinates":[
               [
                  90.386855,
                  23.718213
               ],
               "...
..."
            ],
            "type":"LineString"
         },
         "legs":[
            {
               "summary":"",
               "weight":3303.1,
               "duration":2842.8,
               "steps":[

               ],
               "distance":5250.2
            },
            {
               "summary":"",
               "weight":2536.5,
               "duration":2297,
               "steps":[

               ],
               "distance":4554.8
            }
         ],
         "weight_name":"routability",
         "weight":5839.6,
         "duration":5139.8,
         "distance":9805
      }
   ],
   "cache":false
}
```


If you've hit the above URL for the first time, the `cache` attribute of the json response should show `false`. This means that the response is being served from the original MapBox API. However, hitting the same URL with the same coordinates again will show the cached response and this time the `cache` attribute should show `true`.

## Inspection

Once you've got everything up and running you can inspect the cache via redis insight. To do so, go to the link below while your app server is running:

```
http://localhost:8000/
```

Select the `Browser` panel from the left menu and click on a key of your cached data. It should show something like this:


![Screenshot from 2020-05-25 02-06-24](https://user-images.githubusercontent.com/30027932/82763854-6a74a380-9e2c-11ea-998d-066d25461eca.png)

Also you can play around with the API in the swagger UI. To do so, go to the following link:

```
http://localhost:5000/docs
```

This will take you to the swagger dashboard. Here you can make requests using the interactive UI. Go ahead and inspect how the caching works for new coordinates.

![swagger](https://user-images.githubusercontent.com/30027932/82763965-2f26a480-9e2d-11ea-906b-63c1d25c08a8.png)


## Remarks

All the pieces of codes in the blog were written and tested with python 3.8 on a machine running Ubuntu 18.04. You can find the complete source code of the app [here](https://github.com/rednafi/redis-request-caching).

## Disclaimer

This app has been made for demonstration purpose only. So it might not reflect the best practices of production ready applications. Using APIs without authentication like this is not recommended.

## Resources

* [Http Request Caching with Redis](https://github.com/rednafi/redis-request-caching)
* [Httpx](https://github.com/encode/httpx/)
* [Redis](https://redis.io/)
* [RedisInsight](https://redislabs.com/redisinsight/)
* [FastAPI](https://github.com/tiangolo/fastapi)
