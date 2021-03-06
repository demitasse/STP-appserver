# DAC Speech Transcription Platform
FROM ubuntu:14.04

MAINTAINER Daniel van Niekerk <dvn.demitasse@gmail.com>
LABEL Description="Basic STP platform served over HTTP using Apache"

#Install standard tools from Ubuntu repo
RUN apt-get clean all && apt-get update && apt-get install -y --force-yes apache2 libapache2-mod-proxy-uwsgi uwsgi uwsgi-plugin-python patch python python-bcrypt python-requests python-git sox libsox-fmt-all mp3splt pandoc vorbis-tools

#Configure Apache to forward requests (uWSGI)
COPY stp/install/* /etc/apache2/sites-available/
RUN patch /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf.patch
RUN a2enmod proxy headers

#Setup user and source code for platform
# IF NEEDED: Change FSUID to harmonize with host system user's UID (mounted file system)
ARG UID
ARG GID

RUN groupadd -g $GID dac
RUN useradd -u $UID -g $GID -ms /bin/bash dac
ENV HOME=/home/dac

#Next line should correspond to definitions in `app_server/config/*`
ENV PERSISTENT_FS=/mnt/stp
# Set the following on commandline

WORKDIR $HOME

#Install application server
COPY stp $HOME/source/stp/
RUN chown -R dac:dac source

#Need to set this for GitPython
#See: https://home.regit.org/2014/05/playing-with-python-git/
ENV USERNAME=dac

#Run application server

# You must set following environment variables when running docker
#ENV UID=
#ENV GID=
#ENV SPEECHSERVER=
#ENV APPSERVER=
#ENV SO_SNDTIMEO=

USER root
WORKDIR $HOME/source/stp/app_server
CMD chown -R dac:dac $PERSISTENT_FS && \
    /usr/sbin/apache2ctl start & \
    uwsgi --uid $UID --gid $GID -l 20 -p 10 -z 600 -t 600 --master --enable-threads --plugin /usr/lib/uwsgi/plugins/python27_plugin.so --socket 127.0.0.1:9090 --wsgi-file wsgi.py  --env services_config=config/dispatcher.json
