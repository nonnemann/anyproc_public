## Download the latest Release
You can download the latestest [release](https://github.com/nonnemann/anyproc_public/releases) from this website and start the executable JAR to use AnyProc to create your own toolchains. However, we would like to point out that the releases are published in irregular intervals and therefore may not contain all current feature adjustments or bug fixes.

## Checkout the Development Version
If you wish to checkout the newest version of AnyProc, you can do it by cloning the `main` branch from our directory. Please make sure that you meet the requirements listed below. 

### Requirements
The project is currently running on OpenJDK on Java version 11.0.1 and is built with Maven. Different dependencies are loaded via a pom file. These include:
- GSON
- Commons IO
- Socket IO
- Annotations Java 5

Please make sure that you have imported these dependencies correctly if you wish to checkout the current development build out for yourself.

### Integration of the the ReVize-Server
The integration of the ReVize-Server for the data exchange channel should be done automatically by default. However, in case that the server doesn't start up on your machine, please walk through the following steps. The code for a websocket server, which can be used to distribute [Vega-lite](https:vega.github.io/vega-lite) specifications in realtime across multiple, parallel [revize](https://www.npmjs.com/package/revize)-enabled visualization clients can be found unter [ src/revize_server](https://github.com/nonnemann/AnyProc/tree/main/src/revize_server). Clients that wish to connect to this server need to implement the websocket interface of [ReVize](https://www.npmjs.com/package/revize) version 1.0.1 or higher.

## Prerequisites
ReVize Server uses ```Python 3.8.5```. We recommend setting up an (optional) [virtual environment](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/) for the packages.
It mainly builds on the [socket.io](https://flask-socketio.readthedocs.io/en/latest/) package to create a server for synchronizing vega-lite specifications over websockets and the [flask](https://flask.palletsprojects.com/en/1.1.x/) package for serving the test and debugging environment.

## Installation
To install all dependencies, run the following pip command on a terminal inside this directory:

```bash
pip install -r requirements.txt
```

## Getting started
With all dependencies installed, the server can be launched by running the following command inside this directory:

```
python server.py
```

This will launch a websocket server on ```localhost:5000``` for ReVize clients to connect to. The default namespace for serving vega-lite specs is ```"/test"```.
The testing environment can be reached through the browser on [http://localhost:5000](localhost:5000/).


## Websocket Interface
The server provides a lightweight websocket interface for synchronizing vega-lite specifications. There are three endpoints in total:

### register ```{ id: number }```
Registers a new client in the workflow and shares the latest version and the latest Vega-lite specification with that client.

* The ```id``` property must contain a number, which is expected to be drawn from a random sample.


### update_queue ```{ queue: List<number> }```
Receives an external ordering for clients in the queue and updates the internal queue to that one for calls to next() and previous().

* The ```queue``` property must contain a list of numbers, each representing a vaild ID of a client connected to the server.


### request_queue ```{}```
Responds with the current queue, which is a list of numbers.

### get_next ```{spec: VegaLiteSpec, version: number, source: number}```
Receives an updated Vega-lite specification alongside the version from which the client has derived it, as well as the id of the current client used in the queue.
If that version is equal to the latest version on the server, the specificaction is send out to the next client in the queue, if there is one.

* The ```spec``` property must contain a valid Vega-Lite specification.
* The ```version``` property must contain a number that represents the latest version on which the client worked, when creating the udpated specification.
* The ```source``` property must contain a valid id of the "current" client in the queue.


### get_previous ```{spec: VegaLiteSpec, version: number, source: number}```
Receives an updated Vega-lite specification alongside the version from which the client has derived it, as well as the id of the current client used in the queue.
If that version is equal to the latest version on the server, the specificaction is send out to the previous client in the queue, if there is one.

* The ```spec``` property must contain a valid Vega-Lite specification.
* The ```version``` property must contain a number that represents the latest version on which the client worked, when creating the udpated specification.
* The ```source``` property must contain a valid id of the "current" client in the queue.


### update_spec ```{spec: VegaLiteSpec, version: number}```
Receives an updated Vega-lite specification alongside the version from which the client has derived it.
If that version is equal to the latest version on the server, the specificaction is distributed to all connected clients.

* The ```spec``` property must contain a valid Vega-Lite specification.
* The ```version``` property must contain a number that represents the latest version on which the client worked, when creating the udpated specification.

### disconnect_request ```{}```
Unregisters a client from the workflow.
This endpoint takes has no parameters.
