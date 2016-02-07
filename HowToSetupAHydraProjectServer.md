# How to Setup a Hydra Project Server #

## Prerequisites ##

We'll assume you already have Ruby 1.8.x installed and are on some flavor of linux/unix system. (linux, OS X, BSD, etc)

Also assumes that you have rubygems installed.

## Get Rails and Mongrel ##

```
 sudo gem install rails
 sudo gem install mongrel
 sudo gem install mongrel_cluster
```

## Install Memcached ##

**Warning**: this is probably the most painful part of the install.  Sometimes installing memcached works like a charm, other times you run into dependency issues and it becomes a huge pain.

### Installing Memcached on OS X ###

[Geoffrey Grosenbach](http://nubyonrails.com/) has made this excellent memcached installer script for OS X systems.

_Warning -- again, use the following installer script at your own risk.  Take a peek at it after downloading and do a quick sanity check on it._

How to get and run the script:
```
wget http://topfunky.net/svn/shovel/memcached/install-memcached.sh
chmod 755 install-memcached.sh
./install-memcached.sh
```
Enter your password when it prompts you for sudo rights.

### Installing Memcached on Linux ###

Grab the source code and compile from source:

```
wget http://www.danga.com/memcached/dist/memcached-1.2.4.tar.gz
tar xvzf memcached-1.2.4.tar.gz
cd memcached-1.2.4
./configure && make && sudo make install
```

You might encounter some dependency issues -- i.e. even after installing a dependency, memcached can't find it when compiling.  Ask for help in the forums if you get stuck.

## Install the Memcached Ruby Gem ##

Once you have memcached installed, next you'll need to install the ruby memcache-client gem:

```
 sudo gem install memcache-client
```

## Get the Code ##

If you would like to be able to 'svn up' and pull latest changes, do an svn checkout:
```
 svn checkout http://hydraproject.googlecode.com/svn/trunk/ hydra
```

To export the project (no .svn directories, no easy update mechanism):
```
 svn export http://hydraproject.googlecode.com/svn/trunk/ hydra
```

This will put the code out into a directory 'hydra'.  Cd into the app's config dir:
```
 cd hydra/config
```

## Database Config ##

```
  cp database.yml.example database.yml
  vi database.yml
```

Edit the 'production' entry.

If you're creating a database by hand in MySQL console:

```
CREATE DATABASE hydra;
GRANT ALL PRIVILEGES ON hydra.* TO 'hydra_user'@'localhost' IDENTIFIED BY '2manyseekrets';
# If you need to be able to connect from other (all) hosts, run:
#GRANT ALL PRIVILEGES ON hydra.* TO 'hydra_user'@'%' IDENTIFIED BY '2manyseekrets';
```

_Captain obvious sez, use your own db user / password of course =)._

The database.yml for the above would look like:
```
production:
  adapter: mysql
  database: hydra
  username: hydra_user
  password: 2manyseekrets
  # Uncomment & tweak this line if you're having DB connection issues:
  #socket: /var/lib/mysql/mysql.sock 
```

## Site Config ##

> Edit the config file.  Some of the domain name entries seem redundant; this is in case you want to run the site on a port other than 80 (i.e. foo.org:6969)

```
 cp config.yml.example config.yml
 vi config.yml
```

## Federated Sites Config ##

These are the list of trusted sites that are in your network, or "federation".  _(gotta love the Star Trek reference)_

```
 cp federation.yml.example federation.yml
 vi federation.yml
```

Edit the credentials, note: you do not need to specify (self), i.e. the server you are configuring.  You only specify trusted peer sites.

## Mongrel Cluster Config ##

Create a new mongrel cluster config file in your favorite text editor:

```
vi mongrel_cluster.yml
```

Example contents of your mongrel\_cluster.yml file:

```
---
cwd: /u/apps/hydra
port: "3000"
environment: production
address: 127.0.0.1
pid_file: log/mongrel.pid
servers: 2
```

The above mongrel config assumes the following.

  * Your application's RAILS\_ROOT: /u/apps/hydra
  * Your application's environment: production
  * Ports your app will be started on: 3000 and 3001

If you specified "servers: 6" the ports would be 3000 - 3005.

The more popular your site will be, the more mongrels you will need.  (Note: 6-8 should be plenty for a site with anywhere from 5,000 - 15,000 visitors per day)

## Run the Migrations ##

Change directories back to your rails root and run the migrations:

```
rake db:migrate RAILS_ENV=production
```

## Start the Mongrels ##

Still in rails root, run:
```
mongrel_rails cluster::start
```

You should now have 2 instances running, one at localhost:3000, the other at localhost:3001.  Use apache, nginx, etc. to proxy from port 80 to these mongrels.

## Proxying to Mongrel via Apache/nginx ##

[Time for a Grownup Server](http://blog.codahale.com/2006/06/19/time-for-a-grown-up-server-rails-mongrel-apache-capistrano-and-you/) has a good guide on getting Apache playing nice with mongrel.

**nginx** - the lightweight alternative. [nginx](http://nginx.net/) is a lightweight webserver built for speed and configurability.  Addons are pushed into modules, keeping the core nimble and zippy.

[New nginx Config with Rails Caching](http://brainspl.at/articles/2006/09/12/new-nginx-conf-with-rails-caching)

[Setting up nginx at Planet Argon Hosting](http://docs.planetargon.com/Nginx_Configuration/)

## Adding Sync Cron Jobs ##

See ExampleCronJob for how to setup cron jobs that will automatically sync with other servers in your trusted network.


Best of luck on getting a server setup -- feel free to drop by [the Google Group](http://groups.google.com/group/the-hydra-project) if you hit any snags along the way.