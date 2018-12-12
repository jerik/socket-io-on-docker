# socket-io-on-docker

Building the fundament for this project. 

# Update my node installation ( on OSX )

	brew upgrade node
	> node 9.11.1 -> 11.4.0

In retrospect I think it is not needed to have a node upgrade. It worked for me on another machine where I have node v8.10.0. More important seems to be ``npm``. 

# Update npm 

	> npm WARN npm npm does not support Node.js v11.4.0
	npm install -g npm

# Follwing the guide for socket.io
Based on https://socket.io/get-started/chat/

1. create package.json with specified content
2. add depedencies and stuff to package.json 

```
npm install --save express@4.15.2
...
added 46 packages from 30 contributors and audited 90 packages in 4.344s
found 8 vulnerabilities (3 low, 2 moderate, 3 high)
...
```
I ignored the vulnerabilities

3. create index.json with specified content
4. Run node

```
node index.js
```

This shows: listenning on \*:3000
And calling _localhost:3000_ works as well

# Start with the docker part
Based on https://nodejs.org/en/docs/guides/nodejs-docker-webapp/

1. package.json is already populated by the previous steps
2. copied the index.js to server.js to be inline with the tutorial. 
3. Do the dockerfile stuff

```bash
# used the newest node version 
FROM node:11

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm install --only=production

# Bundle app source
COPY . .

#EXPOSE 8080
# Changed to work with my socket.io server.js
EXPOSE 3000
CMD [ "npm", "start" ]
```

4. Build the docker file 

```
docker build -t jerik/socket-io-app .
```
Behind a proxy you have to use the ``--build-arg`` parameter

```
docker build  --build-arg http_proxy=http://my.proxy.com:8123 --build-arg https_proxy=http://my.proxy.com:8123 -t jerik/sockeit-io-app .
```

Had some warnings, but I ignored them

```bash
Step 4/7 : RUN npm install
 ---> Running in 6fdc11ae52dd
npm WARN socket-chat-example@0.0.1 No repository field.
npm WARN socket-chat-example@0.0.1 No license field.
```

5. Check the docker images

```bash
docker images
jerik/socket-io-app      latest              58646c790614        44 seconds ago      896MB
node                     11                  37f455de4837        3 days ago          894MB
```

6. run the image

```bash
docker run -p 49160:3000 -d jerik/socket-io-app

# Get the docker ID
docker ps
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS                     NAMES
8f0041c7b9c8        jerik/socket-io-app   "npm start"         7 seconds ago       Up 6 seconds        0.0.0.0:49160->3000/tcp   determined_turing

# Print app output
docker logs 8f0041c7b9c8

> socket-chat-example@0.0.1 start /usr/src/app
> node server.js

listening on *:3000
```

7. Try it on the browser. 
	
localhost:3000 does not work. You neeed to get the port of your app that docker mapped. This you can see in the
docker ps output under the column PORTS. In this case  

```
0.0.0.0:49160->3000/tcp
```

So we have to use port 49160 in the browser

```
localhost:49160 
```

Chakka, it works

8. Connect to the docker container 

```bash
docker exec -it 8f0041c7b9c8 /bin/bash

# Here you have to use Port 3000
root@055ec64e7645:/usr/src/app# curl -i localhost:3000
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 20
ETag: W/"14-a12yJ6aqIijHd7B3EQixhLH8XfM"
Date: Tue, 11 Dec 2018 23:05:36 GMT
Connection: keep-alive

<h1>Hello world</h1>root@8f0041c7b9c8:/usr/src/app#
```

# Usefull stuff

Rename the docker container to work smother with docker commands

	docker container rename 8f0041c7b9c8 sia

Stop the docker container

	docker stop sia

Run docker container with a name 

	docker run --name sia -p 49160:3000 -d jerik/socket-io-app


# Continue with the socket.io guide
Based on https://socket.io/get-started/chat/

Rename the container to interact with less keystrokes. And connect to the container

	docker container rename 8f0041c7b9c8 sia
	docker exec -it 8f0041c7b9c8 /bin/bash

First install an editor in container 

	apt-get update
	apt-get install vim-tiny

1. Serving HTML 

```bash
# edit and save according to the guide
vi server.js 
# create the file with the describe content
vi index.html 
exit 
```

restart the node process. I did it with stoping and starting the container

	docker stop sia
	docker start sia

On the browser of the host system check if the new index.html will be servered ``http://localhost:49160``. Worked for me

```bash
docker exec -it 8f0041c7b9c8 /bin/bash
npm install --save socket.io
# edit and save according to the guide
vi server.js
vi index.html
# get the PID of the 'node server.js' process. In my case 16
ps aux 
# kill the node and container, you will be thrown of the container
kill -9 16
```

Restart the container and check the logs

	docker start sia
	docker logs 8f0041c7b98

You should see at the end of the screen ``a user connected``. If you open a new browser tab with ``localhost:49160`` you see more of the log messages. 

I started on a second console a tail of the log mmessage 

```bash 
docker logs -f 8f0041c7b9c8
```

and followed the guide

```bash 
docker exec -it 8f0041c7b9c8 /bin/bash
# edit and save according to the guide (disconnect)
vi server.js
kill -2 16
```

And I have a typo somewhere in server.js which leads to a failed start of the docker container
