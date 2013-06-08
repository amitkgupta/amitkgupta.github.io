---
layout: post
title: "Setting up PostgreSQL with MacPorts for Ruby on Rails development"
date: 2013-05-11 13:39
comments: true
categories: 
---

This post is pretty much just what the title says.  If you develop in Rails, on a Mac, using PostgreSQL as your database and MacPorts as your package management system of choice, it can be hard to get everything set up and going.  This is especially true if you're new to absolutely everything, which was my situation when I first tried to do this.  Part of what was hard was that most of the advice on the Internet was just a bunch of commands to run with little explanation, so it was hard to know what you had to do to customize it for your environment.  So this explanation is hopefully a little more thorough.  However, if you're impatient and just want the **TL;DR**, here it is (with several implicit assumptions):

```bash
sudo port install postgresql92-server
sudo mkdir -p /opt/local/var/db/postgresql92/defaultdb
sudo chown postgres:postgres /opt/local/var/db/postgresql92/defaultdb
sudo su postgres -c '/opt/local/lib/postgresql92/bin/initdb -D /opt/local/var/db/postgresql92/defaultdb'
sudo su postgres
/opt/local/lib/postgresql92/bin/pg_ctl -D /opt/local/var/db/postgresql92/defaultdb/ -l /opt/local/var/db/postgresql92/defaultdb/server.log start
exit
echo 'PATH=/opt/local/lib/postgresql92/bin/:$PATH' >> ~/.bashrc
source ~/.bashrc
rails new my_app --database=postgresql
sed -e 's/username: my_app/username: postgres/g' -i '' my_app/config/database.yml
cd my_app
rake db:create:all
```
<!--more-->
In order to use a PostgreSQL database for development, you'll need, in addition to the PostgreSQL package itself, a PostgreSQL server for your application to talk to.  The PostgreSQL server package has the basic PostgreSQL package as a dependency, so we'll just run the command to install the server and we'll get both.  The **server** package will allow you to run a process that serves your database, and the basic package provides a **client** that your Rails app will use to connect to and interact with (read, write, etc.) the database being served.

Pick the version of PostgreSQL you want to install.  At the time I wrote this, the latest was 9.2.x so we'll go with that:

```bash
sudo port install postgresql92-server
```

You'll likely see the following instructions in the installation output

```bash
To create a database instance, after install do
 sudo mkdir -p /opt/local/var/db/postgresql92/defaultdb
 sudo chown postgres:postgres /opt/local/var/db/postgresql92/defaultdb
 sudo su postgres -c '/opt/local/lib/postgresql92/bin/initdb -D
/opt/local/var/db/postgresql92/defaultdb' 
```

Now in order to start a PostgreSQL server process, it needs an initial database cluster within which you will create your databases.  To do this you need to create a directory for an initial database cluster and tell PostgreSQL to initialize that directory for use with a PostgreSQL database cluster.  PostgreSQL doesn't allow the superuser to initialize a database cluster.  The user used to initialize the database cluster should be one that will exist on any machine that has PostgreSQL, allowing you to collaborate on your Rails app with people using different machines, thus it makes sense to use the 'postgres' user.  The above commands will:

1. Create the directory for an initial default database cluster
1. Change ownership of the directory to the 'postgres' user
1. Initializes the directory as a PostgreSQL database cluster on behalf of the 'postgres' user.

After you do this, you'll see the following instructions in the output:

```bash
Success. You can now start the database server using:

    /opt/local/lib/postgresql92/bin/postgres -D /opt/local/var/db/postgresql92/defaultdb
or
    /opt/local/lib/postgresql92/bin/pg_ctl -D /opt/local/var/db/postgresql92/defaultdb -l logfile start
```

The first will start the server in the foreground, which you probably don't want.  The second will start it in the background, but dump a log file in whatever directory you execute the command, which you don't want either.  You'll also need to start the server as the 'postgres' user, which the above command doesn't do as is, so the solution is to:

1. Switch to the 'postgres' user
1. Issue the above command (with a better path for the log file)
1. Exit from being the 'postgres' user

```bash
sudo su postgres
/opt/local/lib/postgresql92/bin/pg_ctl -D /opt/local/var/db/postgresql92/defaultdb/ -l /opt/local/var/db/postgresql92/defaultdb/server.log start
exit
```

Now when you go to create your Rails project, it will install the `pg` gem for working with PostgreSQL, and it'll configure itself to use the first `psql` (PostgreSQL client) it finds in your `$PATH` environment variable.  Your system comes with one, but you'll want to use the one you just installed.  Assuming you're using a `.bashrc` (or `.bash_profile`) file for initial setup of your bash environment for shell sessions, add 

```bash
PATH=/opt/local/lib/postgresql92/bin/:$PATH
```

to the bottom of the `.bashrc` (or `.bash_profile`) file.  Don't forget to

```bash
source ~/.bashrc
```

for the change to take effect.  

Now that you're done setting up your system for PostgreSQL, you are ready to create and setup a Rails app that uses PostgreSQL.  Start with:

```bash
rails new my_app --database=postgresql
```

The standard `rails new my_app` does a whole bunch of initial setup and file creation for your Rails app.  Adding the `--database=postgresql` flag ensures that your Rails setup includes some PostgreSQL-specific things, such as adding the `pg` gem to your Gemfile, and pre-populating some of the database configuration properties in the `my_app/config/database.yml` file.  We'll need to edit that file a little.  Go to `my_app/config/database.yml` and change the username for the development and test databases to 'postgres'.  What this does is ensure that when your Rails app uses the PostgreSQL client to try to access the database cluster served by your PostgreSQL server, it does so with the credentials of the user who owns that cluster, namely the 'postgres' user.

While you're in that file, you can get rid of the section for the production database altogether if you're deploying to Cloud Foundry or Heroku, since they will overwrite whatever you have there anyways.

Finally, create the development and test databases that your Rails app will use.  (These databases will be created within your default cluster). 

```bash
rake db:create:all
```

Now you're totally ready to go!
