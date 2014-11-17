---
layout: post
title: Using Docker to launch Flask app
---

For this post you'll need Docker. I'm running on Arch, `sudo pacman -S docker` depending on your OS you may have more legwork to do.

1. install docker
1. `sudo systemctl start docker` or `sudo service docker start`
2. `usermod -a -G docker you`
3. Log out/in to get the group
	* you can verify with `groups | grep docker`
4. run docker pull fdavis/utopic-flask-tut
5. Go to your checkout of your flask app (run.py will need host='0.0.0.0' if main starts with via flask)
5. `docker run -i -t -P -v $(pwd)/flaskapp:/opt/flaskapp -p 5000:5000 fdavis/utopic-flask-tut  /opt/flaskapp/run.py`

You should be able to see your flask app in localhost:5000 :smiley:


# Setting up a docker box

I started to automate this build process, hopefully I'll complete it soon. Until then if you need your own docker box setup, or mine does not work, we can follow the steps [from my Dockerfile](https://github.com/fdavis/dockerbling/blob/master/dockerfiles/utopic-flask/Dockerfile)


start with a docker utopic box (if you chose another version or distro you may have to change package names as appropriate)

~~~ bash
docker run -i -t -p 5000:5000 ubuntu:utopic bash
~~~

You can always use `docker help` for a list of commands and `docker help run` for help on a specific command. I use `-i -t` for an interactive tty. `-P` publishes all known ports, which isn't helpful in this case, but if you use my Dockerfile it would forward ports 5000 and 80 (Use `docker port` too see the exact host port number.) This way we get a bash prompt on the docker container so we can install the requisite software. Run these:

~~~ bash
apt-get update
apt-get install -y build-essential python python-dev python-pip python-mysqldb libmysqlclient-dev supervisor libmemcached-dev memcached python-memcache
pip install flask flask-login flask-mail sqlalchemy flask-sqlalchemy flask-wtf flask-migrate tornado flask-cache simpleencode pdfminer flask-admin flask-security
~~~

Now if you send `CTRL+D` you will leave the guest container's bash prompt, and `docker ps` will no longer show that box because it has stopped. Docker stops the box by design because its meant for running a single process like a tiny apache server. You should use `docker ps -a` so you can get that last box's container id which you'll need for the next command. Run `docker commit $YOUR_HASH utopic-flask-tut` to create an image from the container you've been working with. Now you can launch a new container from the previous container's state using:

## The docker run command for /opt/flaskapp/run.py

~~~ bash
docker run -i -t -P -v $(pwd)/flaskapp:/opt/flaskapp -p 5000:5000 utopic-flask-tut /opt/flaskapp/run.py
~~~

The volume share requires the full path so I just prefix with `$(pwd)` to make it easier. So `-v $(pwd)/flaskapp:/opt/flaskapp` is to share my /home/fdavis/git/flaskapp/ to the docker container's /opt/flaskapp, 