docker-nginx-sftp
=================
[![Docker Pulls](https://img.shields.io/docker/pulls/bdominik/docker-nginx-sftp.svg)](https://hub.docker.com/r/bdominik/docker-nginx-sftp/) [![Docker layers](https://images.microbadger.com/badges/image/bdominik/docker-nginx-sftp.svg)](https://microbadger.com/images/bdominik/docker-nginx-sftp) [![Github Stars](https://img.shields.io/github/stars/theomega/docker-nginx-sftp.svg?label=github%20%E2%98%85)](https://github.com/theomega/docker-nginx-sftp/)

Purpose
-------
The image provides an http server which serves static files. The static files
can be modified using sftp. This is done by launching both, an http server
(nginx) and an sftp server (openssh-sftp) inside the docker container. The
static files can be persisted on a volume to make the surive restarts of the
container.

This container allows to host a website on a docker host where some third party
(who knows the credentials) can modify the website via any sftp client. There
are sftp clients for nearly all operating systems.

Usage
-----
To run this container use

     docker run -ti -e USER=myuser -e PASSWORD=mypassword -p 2222:22 -p 8888:80 -v
     /tmp/data_for_container:/data -v /tmp/keys_for_container:/etc/ssh/keys/
     bdominik/docker-nginx-sftp:latest

You can then using any sftp client connect to localhost:2222 and put files into
the /webroot folder. You need to log in with the provided username and password.
Files from the /webroot folder are served then via http on
http://localhost:8888.

Configuration
-------------
There are two environment variables which you have to provide when launching the
container to specify the username and the password which can then be used to log
into the sftp server. These are called `USER` and `PASSWORD.

The container listens on two ports, 22 for the sftp server and 80 for the http
server. The container intentionally does not provide an SSL/HTTPS interface as
this can be handled using other docker container easily.

Volumes
-------
You can mount two volumes to the docker-images as in the example above:

  * `/data` is the folder which contains the static files which are served from
    the nginx http server and can be modified via sftp.
  * `/etc/ssh/keys` is the folder which contains the ssh host keys for the sftp
    server. If the host keys are not existing, they are created on start. If you
    don't mount this volume, you will get new ssh host keys everytime the
    container launches which will lead into connection error for the users of
    the sftp server.

Internals
---------
The internals of this image are quiet straight forward. The container is based
on alpine linux and contains the following additional packages:
  * `openssh-server` and `openssh-sftp-server` to provide the sftp server
  * `nginx` to provide the http server
  * `supervisord` to orchestrate the two processes (nginx + openssh) and keep
    them running.

The packages are configured using the configuration files in this repo.
Supervisord is configured to fail the whole container is either of the two
processes fail. All the logging goes to the docker output, so you will see both,
the nginx access log and the sftp connection output.

A small python script called `docker_kill.py` is used as an eventlistener for
supervisord. The listener automatically kills the whole supervisord (and with
this the whole container) if one of the subprocesses (nginx and sshd) fail.

Anti-Pattern
------------
To have two processes in a docker container is an anti-pattern, so think twice
before using this image. The normal way to do this would be to have two separate
docker images, one for the sftp server and one for the http server and have a
shared volume. This container takes another approach for experimentation
purposes.
