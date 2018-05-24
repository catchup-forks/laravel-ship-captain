FROM python:slim

RUN pip install --upgrade --no-cache-dir awsebcli
RUN apt-get -yqq update && apt-get -yqq install git-all

RUN mkdir root/tmp_ssh
COPY ./ssh/. /root/.ssh/
RUN cd /root/.ssh && chmod 600 * && chmod 644 *.pub

WORKDIR /var/www
