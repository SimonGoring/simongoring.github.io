---
layout: page
res_class: Development
title: Containers in the Cloud!
concept: Deploying a simple node.js application to Heroku using Docker containers.
---

<style>
h1:
</style>

By: Simon Goring


# Moving a Node.js App to Heroku with Docker

> *This application is all stored in a GitHub repository: https://github.com/SimonGoring/node_docker.  Unfortunately I didn't initialize the git repo right away, so you can't follow along easily with the commits, but the code is all there, including this file.*

I've been working with `node.js` as a tool to [rebuild the Neotoma Database's API](http://neotomadb/api_nodetest).  As part of that process we'd like to build a self-contained unit that can be used to do development locally, but also to serve the container on the remote server.

While I'm familiar with Docker and node, I haven't put the two together, so this is a first attempt at putting it all together to generate an app up on AWS.  I am following documentation on the [`node.js` website](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/).

## Setting up node.js and express

### Creating your Repo

This should have been the first thing I did, but it wasn't.  With whatever version control system you want generate a new repository and clone it locally.  Later on we're going to use [Heroku](http://heroku.com) to deploy the application.  One of the options with a Heroku instance is to link it to your GitHub (or other) repository, so that new pushes get automatically deployed to Heroku.  This way you just have to commit and push once to get the benefit of an updated instance *and* version control.

As mentioned above, my repo for this project is at https://github.com/SimonGoring/node_docker.

### Creating the package

The tutorial asks us to start with a `package.json` file, but I went with using `npm install` at the command line, and then hand editing so that I got something like this:

```json
{
  "name": "node_docker",
  "version": "0.1.0",
  "description": "Trying out a simple node app with Docker.",
  "main": "server.js",
  "dependencies": {},
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Simon Goring",
  "license": "MIT"
}
```

To add the `express` dependency I used `npm install express --save` which resulted in a `package.json` that looked like this:

```json
{
  "name": "node_docker",
  "version": "0.1.0",
  "description": "Trying out a simple node app with Docker.",
  "main": "server.js",
  "dependencies": {
    "express": "^4.15.3"
  },
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Simon Goring",
  "license": "MIT"
}
```

This looks pretty close to what was in the `node.js` tutorial, the differences are the `name` and general descriptive tags.  I also don't have the `scripts` tag set up properly.  `npm start` automatically runs `server.js` when it runs, and I went in and deleted the `scripts:test` line.  I'll work on tests later, so for completeness I went and edited the `scripts` section.  So now we're at a `package.json` file that looks like this:

```json
{
  "name": "node_docker",
  "version": "0.1.0",
  "description": "Trying out a simple node app with Docker.",
  "main": "server.js",
  "dependencies": {
    "express": "^4.15.3"
  },
  "devDependencies": {},
  "scripts": {
    "start": "node server.js"
  },
  "author": "Simon Goring",
  "license": "MIT"
}
```

### Writing up the Server JavaScript

The basic app is in a file called `server.js` in the main directory.  I was reading some documentation that talks about keeping things out of the root directory.  In general I like this idea, I think that having all these files in the root directory makes things messy when you're looking at a GitHub repo, but that's maybe just me.

Regardless, I basically copied the `server.js` file from [the tutorial](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/) with some minor changes:

```js
'use strict';

// We're using the express library we pulled in using npm:
const express = require('express');

// Constants (note this will get weird later)
const PORT = 8080;

// We start an express app:
const app = express();

// At the root, create a `send` element for the `res`ponse.  There is no 
// option to pass in a `req`uest at this time.

app.get('/', function (req, res) {
  res.send('<h1>Hello world</h1>\nAnd a special hello to Simon!');
});

// Then listen to the port to see if there are any `get` calls to the root (with the response defined above):
app.listen(PORT);
console.log('Running on http://localhost:' + PORT);
```

A couple points about the code as it is.  (1) The use of `'use strict'` means that code is executed in as literal a manner as possible.  
  * Variables can only be created when declared.  Mis-typing a variable name doesn't create a new variable, it throws an error.
  * Assignments to properties that can't be overwritten throws an explicit error.
  * It prevents non-unique parameters.

There's good documentation on the [Mozilla Developer's strict mode page](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode).

I've heavily commented it, for the purposes of this document, but that's not necessary. You can also change the function in the `app.get` command to return anything you want.  You can even add more endpoints, as I'm [slowly doing in the Neotoma API](https://github.com/NeotomaDB/api_nodetest/blob/dev/routes/index.js) using the `express.Router`.

### Running the Server code

The app only really prints out "Hello World" to the browser, and then logs "Running. . . " to the log.  We can test it out by running the same command that we used in our `scripts:start` element:

```bash
node server.js
```

(this works for me)

## Setting up Docker

Docker is a container system for your computer.  Think of it as a computer inside a computer.  It's similar to a Virtual Machine, except it uses your existing computer's OS, so it's more lightweight.  It lets you package up the components needed to develop an app in a clean environment, so they you don't wind up with conflicts between one project and another, and it lets you clearly define the components, so that installation in a new system is not affected by different environment variables, packages or dependencies.  This makes Docker very useful for projects where you are either developing multiple projects, projects where you might be deploying to a different system, or projects where you are collaborating with a number of colleagues using different systems.

You need to install Docker before you can use it.  I will not go over that.  You can read the [Installing Docker](https://docs.docker.com/engine/installation/) documentation from Docker. 

#### Briefly, the Dockerfile

To run Docker, you need a `Dockerfile` from which to initialize the container.  Docker has good documentation on [`Dockerfile` best practices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) that is worth reading.  We use the `FROM` argument to define the location of the `image` that we'll be using as the basis of our container, we tell Docker what to `RUN` as it's installing things, we tell it where to `COPY` the files it will be using, we tell it how to `RUN` npm to set up the project locally, and then we copy, `EXPOSE` a port, and then call the `CMD` to actually start the project.

```bash

# Define the docker hub image: https://hub.docker.com/_/node/
FROM node:alpine

# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Install app dependencies
COPY package.json /usr/src/app/
RUN npm install

# Bundle app source
COPY . /usr/src/app

EXPOSE 8080
CMD [ "npm", "start" ]
```

#### Again the Dockerfile

So, we create a `Dockerfile` in our local directory and we'll use it to start our app.  When it starts it will download the node image, then in the container (which Docker creates) we create a `/usr/src/app` folder, which is where the source code for the app is going to go.

The `Dockerfile` then sets the `WORKDIR` to this folder, and moves the *local* `package.json` file from the current directory into the new `/usr/src/app/` folder.  Once that's done, in the `WORKDIR` it runs `npm install` to initialize the `node.js` application we've written above.

Once the application is initialized then we copy all the files in the current directory (`.`) into the `/usr/src/app` folder.  Given this, I would probably change some of these parameters around if we used a nested directory structure.

Finally, the `node.js` app, by default, is broadcast through port `8080`.  This means that we need to let `localhost:8080` broadacst to the pasten machine (ours).  Finally we us the `CMD` `npm start`, but it's concatenated.

#### And the .dockerignore
The [Dockerfile Best practices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) says we should also add a `.dockerignore` file.  Here we'll put our node files:

```bash
node_modules
npm-debug.log
```

### The Moment of Truth

With the `Dockerfile` and the `.dockerignore` files created, the next thing to do is to *build* the Docker container.  We build the container and attach it to the system so that we can run it when we're ready.  We need to give it an informative name so that we know where it is, so we *tag* it using the `-t` parameter:

```bash
docker build -t simon/node_docker .
```

As this is happening you should see a whole bunch of output that looks something like this:

```
Sending build context to Docker daemon  11.78kB
Step 1/8 : FROM node:alpine
alpine: Pulling from library/node
79650cf9cc01: Pull complete 
2d271477e3b2: Pull complete 
49eae82ca692: Pull complete 
Digest: sha256:ec27361dcb1a1467f182c98e3e973123fda92580ef7b60b17166f550124a98a3
Status: Downloaded newer image for node:alpine
 ---> 2a7d8107cda5


Removing intermediate container 8f84cc1f9584
Successfully built 0cb6b0d27398
Successfully tagged simon/node_docker:latest
```

This should mean that everything's been loaded into a container.  We can check to see if it's there using `docker image`.  You should see your container, with the tag, listed.  It's there, in the Docker app, but it's not actually running right now.

To run a Docker container we need to use the command `docker run`.  This will tell Docker to set up the files as defined in the `Dockerfile`.  But there are some other things we need to do.  First, in the `Dockerfile` we expose a port (`8080`). Because Docker is running inside the machine, there's no guarantee that `8080` is available.  We need to bind the Docker port to a port on the parent machine using the `-p` flag (`-p 12345:8080`) so it says that to get to the `8080` port on the container we have to go in through our own `12345` port.  Then we want to be clear that the name of the app project we're going to be running is `simon/node_docker`, since we may have multiple Docker containers on our computer.

```bash
docker run -p 12345:8080 -d simon/node_docker
```

Running this command gives you a long hash that is the container ID.  You can copy this to a text file, or find your container ID and container status at any time using [`docker ps`](https://docs.docker.com/engine/reference/commandline/ps/):

```bash
$ docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
a7c2debade1a        simon/node_docker   "npm start"         9 minutes ago       Up 8 minutes        0.0.0.0:12345->8080/tcp   loving_einstein
```

You can take the `CONTAINER ID` (in this case `a7c2debade1a`) and use it to  return the container logs:

```
$ docker logs a7c2debade1a

npm info it worked if it ends with ok
npm info using npm@4.2.0
npm info using node@v7.10.0
npm info lifecycle node_docker@0.1.0~prestart: node_docker@0.1.0
npm info lifecycle node_docker@0.1.0~start: node_docker@0.1.0

> node_docker@0.1.0 start /usr/src/app
> node server.js

Running on http://localhost:8080
```

In this case, it tells us that the container is running fine.  As a lark, I am going to stop Docker, remove the container, make an edit to the server.js file to break it, and then start the process again:

```bash
$ docker stop a7c2debade1a
a7c2debade1a
$ docker rm a7c2debade1a
a7c2debade1a
$ echo "
console.lug(asa);" >> server.js
$ docker build -t simon/node_docker .
$ docker run -p 12345:8080 -d simon/node_docker
```

Once this is done everything should look fine, but if you take the has and look at your `docker logs`, you'll see it's a huge mess:

```
$ docker logs 40b1
npm info it worked if it ends with ok
npm info using npm@4.2.0
npm info using node@v7.10.0
npm info lifecycle node_docker@0.1.0~prestart: node_docker@0.1.0
npm info lifecycle node_docker@0.1.0~start: node_docker@0.1.0

> node_docker@0.1.0 start /usr/src/app
> node server.js

/usr/src/app/server.js:17
\nconsole.lug(asa);
^
SyntaxError: Invalid or unexpected token
    at createScript (vm.js:53:10)
    at Object.runInThisContext (vm.js:95:10)
    at Module._compile (module.js:543:28)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.runMain (module.js:605:10)
    at run (bootstrap_node.js:427:7)
    at startup (bootstrap_node.js:151:9)

npm info lifecycle node_docker@0.1.0~start: Failed to exec start script

.
.
.
```

Gross.  Don't make mistakes, and when you do, go back and look at your logs.  I cleaned this error up in `server.js`, and then went back and re-ran the container.

### Testing the Implementation

So, we've got the container running in the background, we know it's running well, now we want to actually test it.  In the `docker logs` we see that the Docker container tells us it's running on `localhost:8080`, but that's a trick.  It is running on `8080` within the container, but remember that in our `docker run` command above we assigned the container port (`8080`) to our host's port of `12345`.  So, we can either navigate to `http://localhost:12345`, or use `curl`:

```bash
curl -i localhost:49160
```

Using the `i` flag in this way gives us both the **Hello World** response defined above and the header information.

## Releasing the App

One of the advantages of Docker is being able to deploy the application to any platform in a consistent way.  The platform (AWS, Heroku, Azure) manages the OS and Docker manages the required files.  You can have multiple Docker containers, with different dependencies on a single platform as long as the ports don't overlap.

In this example I've chosen [Heroku](https://heroku.com/) as the platform.  It provides free *microinstances* for public repositories.  With my heroku account I can use the command line argument:

```
heroku start
```

to start a microinstance.  In my case the microinstance returns:

```
Creating app... done, ⬢ still-brook-77635
https://still-brook-77635.herokuapp.com/ | https://git.heroku.com/still-brook-77635.git
```

We also need to install some auxillary packages for Heroku:

```bash
heroku plugins:install heroku-container-registry
heroku container:login
```

Once this is all done, we should be able to push it up to the web and have it compile up there.

```
heroku container:push web
```

It's worthwhile to note that the first time I pushed this, as configured above, it failed with the error message:

>  Web process failed to bind to $PORT within 60 seconds of launch.

I found [a question on StackOverflow](https://stackoverflow.com/questions/15693192/heroku-node-js-error-web-process-failed-to-bind-to-port-within-60-seconds-of) that helped me fix the error.  In the `server.js` file I created I changed the line:

```javascript
app.listen(PORT);
```

to:

```
app.listen(process.env.PORT || 8080);
```

This works because `process.env.PORT` pulls the process's environment variable `PORT`, and, if it doesn't exist, it sets the port to `8080`.  Part of the reason this works is that Heroku defines a port for you, so you can't count on `8080` being available, but on your local machine there is unlikely to be an existing `PORT` environment variable, and so `8080` becomes the port of choice.

So, changing this parameter, commiting and then pushing back to Heroku with:

```bash
heroku container:push web
```

then allows the app to run in the cloud.  You can [see my incredibly basic app here](https://still-brook-77635.herokuapp.com/).

# Conclusion

So now we have a workflow to develop an app locally inside a container that keeps it quarrantined from the rest of our computer, and because the container is self describing, that lets us share the work with anyone we want to, with little or no fear that their system will over-ride our requirements.  We also can then use a web service (Heroku) to push the container up and serve it from the web.  Great stuff everyone!

I'm going to keep working on this, but if you have comments, suggestions or want to fix mistakes, please feel free to fork [this repo](https://github.com/SimonGoring/node_docker), or raise issues in the [repository issue tracker](https://github.com/SimonGoring/node_docker/issues).