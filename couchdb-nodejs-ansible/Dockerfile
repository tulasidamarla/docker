# Pull base image.
FROM opensuse:13.2

MAINTAINER Tulasi Ram ram.damarla@gmail.com

RUN zypper update -y

#
# Python
#
RUN zypper install -y python python-devel python-pip python-virtualenv

#
# Node.js and NPM
#
RUN zypper install -y nodejs npm git
RUN ln -s /dev/null

#
# Grunt, Bower, Forever
#

RUN npm install -g grunt \
        && npm install -g bower \
        && npm install -g forever

#
#CouchDB
#
RUN useradd -m couchdb \
        && zypper update -y \
        && zypper install -y curl which
VOLUME /var/lib/couchdb
RUN zypper install -y couchdb
RUN sed -i'' 's/bind_address = 127.0.0.1/bind_address = 0.0.0.0/' /etc/couchdb/default.ini
EXPOSE 5984

#
# Ansible
#

RUN zypper update -y && zypper install -y ansible

#start couchdb
CMD ["/usr/bin/couchdb"]


