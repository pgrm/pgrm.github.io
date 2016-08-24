---
title: How to run Meteor in Cloud9
date: 2014-03-22 00:00:00 Z
tags:
- Online-Development
- Cloud9
- Meteor
redirect_from:
- "/how-to-run-meteor/"
---

I've written a long question about this on Stack Overflow ([Meteor with cloud9](http://stackoverflow.com/questions/22561328/meteor-with-cloud9/)). Basically everything is written there, I just wanted to summarise it here and skip the useless things I found out.

## Initial Problem:
Meteor can't be installed on cloud9, as it is described on the website, you need to clone it out from the [repository](https://github.com/meteor/meteor/tree/master). In the terminal:

    cd
    git clone https://github.com/meteor/meteor.git
    cd meteor
    git checkout master
    ./meteor

Now you have the lastest release with all dependencies downloaded. Change to your working directory, where you use meteor to proceed...

## Second Problem:

### IP-Address and Port Bindings:

Running

    ~/meteor/meteor

will show you an error like:

```
 Cloud9  Error: you may be using the wrong PORT & HOST for your server app
         Node: use 'process.env.PORT' as the port and 'process.env.IP' as the host in your scripts. See also https://c9.io/site/blog/2013/05/can-i-use-cloud9-to-do-x/
```
Meteor already uses the `PORT` environment variable, but instead of `IP` it uses `BIND_IP` so, by calling

    export BIND_IP=$IP

we should solve it, but that's not enough. There is a bug in meteor, which creates a proxy on `localhost` instead of `BIND_IP`.

To fix this we go to:

    cd ~/meteor/tools/
    vim run-proxy.js

In line 94 (`:94`) you see:

    self.server.listen(self.listenPort, function () {

There is a second parameter, the `BIND_IP` missing. Replace the line by the following:

    self.server.listen(self.listenPort, process.env.BIND_IP, function () {

While we are in this file, there are two more errors on lines 170 (`:170`) and 181 (`:181`). The lines look the same:

    target: 'http://127.0.0.1:' + self.proxyToPort

Replace both of them with:

    target: 'http://' + process.env.BIND_IP + ':' + self.proxyToPort

Now Meteor is running from the correct IP but not the correct PORT. Even you can find in many posts that Meteor uses a `PORT` environment variable to define the port, this isn't entirely true anymore. Luckily for us, Cloud9 already exports a port, which should be used, into the environment variable `PORT`. So all we need to do, is assign it on one place:

    cd ~/meteor/tools
    vim run-all.js

In line 24 (`:24`) you can see:

    var listenPort = options.port;

We'll replace `options.port` by `process.env.PORT`. Replace the line by the following:

    var listenPort = process.env.PORT;

Going back to your project and running

    ~/meteor/meteor

you should have now a problem with the MongoDB...

## Last Problem:

### MongoDB

Now I didn't figure out how to run MongoDB inside Cloud9, but it isn't necessary. You can instead create a free account on [MongoHQ](https://www.mongohq.com/) and have your development database there. 512MB should be enough for most of the projects, it's definitely enough for me, so far. (They don't pay me anything for writing this.)

Once you have your database, you can see the connection string in your admin panel:

    mongodb://<user>:<password>@oceanic.mongohq.com:10014/<your-db-name>

Meteor uses the environment variable `MONGO_URL` for this, so just run:

    export MONGO_URL=mongodb://<user>:<password>@oceanic.mongohq.com:10014/<your-db-name>

## Conclusion:

After you have modified `meteor/tools/run-proxy.js` and added `process.env.BIND_IP`, you might want to add the following two lines to your `.bashrc`:

    export BIND_IP=$IP
    export MONGO_URL=mongodb://<user>:<password>@oceanic.mongohq.com:10014/<your-db-name>

Now open a new terminal (or run `source ~/.bashrc`) and you are good to go. Meteor will from now on work with the command

    ~/meteor/meteor

To access your, successfully running, application, you can go to `https://<project-name>.<user-id>.c9.io`.
