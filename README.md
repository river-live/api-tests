# Load Testing the River Infrastructure

To load test the deployed River infrastructure on AWS, we used the following setup:

- two c5.large EC2 instances
- the artillery.io testing framework
- the "demo" server (in our server repo under the `demo` branch)

## Overview

To simulate a significant load on our River infrastructure, we used these conditions:

- adding 100 new users (connections) per second
- upon connection, each new user:
  - subscribes to a channel
  - holds the connection for 10 minutes
  - subscribes to one more channel
  - disconnects

The server treats the channel as a "presence" channel, meaning that each time a new subscription happens, a websocket message announcing the new user is sent to all the other users subscribed to that channel.

As the number of connections grew, the rate of connections slowed to more like 15 per second, which means that in the last second (connecting users 19985 thru 20000) of connections being made:

- connection 19985 results in 19984 messages being sent
- connection 19986 results in 19985 messages being sent
- ... etc.
- connection 20000 results in 19999 messages being sent.

Which means in the last second the server sent about 300,000 messages.

River was able to handle this load without any errors.

![Load Test](images/load-test.png)

## EC2 setup

- Ubuntu 18.04
- c5.large

Once a terminal is open with a ssh connection, install the LTS version of Node.js

```
curl -sL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

- install artillery.io

```
npm install -g artillery
```

This will throw a permissions error which can be fixed by changing the directory ownership

```
whoami
sudo chown -R $USER /usr/lib/node_modules
```

Change the limit for number of files open at a time (each WebSocket connection creates a new file):

```
ulimit -n 65536
```
