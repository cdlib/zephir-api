## Introduction

The Zephir API is a support service for Zephir, the HathiTrust metadata management system. 
The API is a Sinatra-based web API that allows access to data not available through
normal Zephir exports (e.g. pre-clustered item metadata for an object in the HathiTrust 
repository. This application is only meant to be run in the context of Zephir.

When available, the following content types are supported:

* XMl (text/xml)
* JSON (application/json)

## Readme and Information
* README (information about the code)
* API (information about the API provided by code)
* /docs (generated code documentation. See below

## Access Services

Documentation: http://zephir.cdlib.org/api/documentation

Item API: http://zephir.cdlib.org/api/item/{HT_ID}

For example:
* http://zephir.cdlib.org/api/item/mdp.39015012078393 (default to XML output)
* http://zephir.cdlib.org/api/item/mdp.39015012078393.xml
* http://zephir.cdlib.org/api/item/mdp.39015012078393.json

## Start/Stop Services

The Zephir API service is provided through the Apache Web server. Start or stop the Apache Web server to enable or disable the service.

#### Start at Server Reboot
```
sudo su - htmm
sudo init 6
```
#### Through Monit (Start/Stop/Monitor)
Start monit daemon:
```
monit -c /apps/htmm/.monitrc 
``` 
Start all services:
```
monit -c /apps/htmm/.monitrc start all
```
Stop all services:
```
monit -c /apps/htmm/.monitrc stop all 
```
Check status:
```
monit -c /apps/htmm/.monitrc status
```
Note: Provide the control file /apps/htmm/.monitrc to the monit command. Otherwise monit will use it’s default which is /etc/monit.conf.

#### Through Apache httpd Start/Stop
Start httpd
```
/usr/sbin/httpd -k start -f  /apps/htmm/cdl-env/httpd-passenger.conf
```
Stop httpd
```
/usr/sbin/httpd -k stop -f  /apps/htmm/cdl-env/httpd-passenger.conf
```

## Requirements

This code has been run and tested on Ruby 2.5.7 and Sinatra 2.0.7.

* Ruby 2.5.7
* Sinatra 2.0.7
* Bundler
* Apache
* mod_passenger for Apache
* sqlite3
* mysql2
* Gem dependencies
* zephir-services-cdl-env

## Installation

* Role account: htmm
* Home directory: /apps/htmm

### 1. Create SSH keys for the htmm account

Create the .ssh directory for the htmm account if it does not exist. The .ssh directory permissions should be 700 (drwx------). 
Create a public and private key sets for the htmm account if needed. 
The permissions for the public key (.pub file) should be 644 (rw-r-r--). 
For the private key (id_rsa) the permission should be 600 (rw------).

Add the public key to the authorized_keys file of d2d-zephir2-dev and/or d2d-zephir2-stg for passwordless scp commands listed in step 2.
```
    cd ~
    mkdir .ssh
    chmod 700 .ssh
    cd .ssh
    ssh-keygen
```

### 2. Get the Git deploy keys and configuration file

Copy over the zephir-api and htmm-env repository deploy keys (private and public key pairs) and the configuration file from zephir-dev or zephir-stg.
The keys and the configuration file are under the .ssh directory of the htmm account:
* id_rsa_zephir-api
* id_rsa_zephir-api.pub
* id_rsa_cdl-env
* id_rsa_cdl-env.pub
* config

Make sure the key files have the proper permissions:
* Private key permission: 600
* Public key permission: 640

Change file permissions if needed.

The SSH configuration file should have the following entries:
```
/apps/htmm/.ssh/config

Host github.com-cdl-env
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_cdl-env

Host github.com-zephir-api
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_zephir-api
```

### 3. Install the application and prerequisites and dependencies 

A script was created to automate the installation and configuration the following items:
* Apache
* Monit
* Ruby
* sqlite
* mysql
* Zephir-API
* htmm-env/cdl-env - a suite of configurations and tools to compliment the Zephir system
 
Readme:
https://github.com/cdlib/htmm-env/blob/12579_zephir_linux2/hosts/zephir/linux2_migration/README.md

Script:
https://github.com/cdlib/htmm-env/blob/12579_zephir_linux2/hosts/zephir/linux2_migration/zephir_linux2_migration.sh

#### 3.1 Install the Zephir-API manually 

```
sudo su - htmm
cd ~
mkdir apps
cd apps
git clone git@github.com-zephir-api:cdlib/zephir-api.git
```

#### 3.2 Install htmm-env/cdl-env manually

* 1st: clone htmm-env repository a suite of configurations and tools to compliment the Zephir system. 
```
sudo su - htmm
cd ~
git clone git@github.com-cdl-env:cdlib/htmm-env.git cdl-env
```
* 2nd: create symbolic links for the dot files
  * .bash_profile
  * .bashrc
  * .forward

#### 3.3 Install prerequisites and dependencies 

* Ruby Manager and plugins
* Dependencies for Ruby
* Ruby 2.5.7
* Package dependency manager Bundler
* Prerequisites
* Apache with Passenger
* Process management infrastructure

```
  ## Check out rbenv and plugins into ~/local/bin/rbenv
  git clone https://github.com/rbenv/rbenv.git ~/local/bin/rbenv
  git clone https://github.com/rbenv/ruby-build.git ~/local/bin/rbenv/plugins/ruby-build

  ## Install dependencies for Ruby
  sudo yum install gcc gcc-c++
  sudo yum install openssl-devel readline-devel zlib-devel

  ## Install Ruby 2.5 for htmm
  rbenv install 2.5.7

  rbenv global 2.5.7
  rbenv rehash

  ## Install the package dependency manager Bundler
  gem install bundler 

  ## install apache
  sudo yum remove httpd*

  sudo yum install httpd
  sudo yum install httpd-devel
  sudo yum install apr-devel
  sudo yum install apr-util-devel
  sudo yum install libcurl-devel

  ## Install Passenger module
  gem install passenger -v=6.0.4 
  passenger-install-apache2-module

  sudo yum install https://kojipkgs.fedoraproject.org/packages/monit/5.14/1.el6/x86_64/monit-5.14-1.el6.x86_64.rpm

  sudo yum install sqlite-devel
  sudo yum install mysql-devel

  ## Install MySQL 5.7
  ## Step 1: Install the MySQL Yum Repository
  sudo yum localinstall /net/homesrv/repo/pkgs/mysql/mysql80-community-release-el7-1.noarch.rpm

  ## Step 2: Enable the MySQL version
  sudo yum-config-manager --enable mysql57-community
  sudo yum-config-manager --disable mysql80-community

  ## Step 3: Use yum to install/update mysql packages
  sudo yum install mysql-community-server
```

### 4. Install gem dependencies with bundler

```
cd ~/ apps/zephir-api/
bundle install
```

### 5. Setup database configuration file

Configuration file: zephir-api/config/database.yml

Include the test database instance listed below. 
Setup the development and production database instances accordingly.

```
test:
  adapter: sqlite3
  database: test/db/test.sqlite

development:
  adapter: mysql2
  database: htmm
  username: htmmro
  password: xxxxxx
  host: rds-d2d-htmm-dev.cmcguhglinoa.us-west-2.rds.amazonaws.com
  port: 3306
```

Note: Add development and production database instances accordingly.

## Tests

#### 1. Confirm installation with the default Ruby Minitest framework. 

```
cd /apps/htmm/apps/zephir-api

rake test

Output:
-- create_table(:zephir_filedata, {:id=>false})
Run options: -v --seed 61892

Running:

TestAPI#test_errors = 0.02 s = .
TestAPI#test_get_documentation = 0.00 s = .
TestAPI#test_get_ping = 0.01 s = .
TestItemAPI#test_get_item_errors_conditions = 0.01 s = .
TestItemAPI#test_get_item_content_negotiation = 0.02 s = .
TestItemAPI#test_get_item = 0.01 s = .

Finished in 0.080585s, 74.4557 runs/s, 260.5948 assertions/s.

6 runs, 21 assertions, 0 failures, 0 errors, 0 skips
```

#### 2. Confirm installation with WEBrick (web server):

Start the web server
```
rackup

[2015-07-28 16:42:10] INFO  WEBrick 1.3.1
[2015-07-28 16:42:10] INFO  ruby 2.2.2 (2015-04-13) [x86_64-linux]
[2015-07-28 16:42:10] INFO  WEBrick::HTTPServer#start: pid=3223 port=9292
```

From the same server run wget to retrieve the Zephir API documentation page:
```
wget 127.0.0.1:9292
```

The WEBrick Web server should have the following log entries reflecting successful access of the documentation page (with status '200'):
```
127.0.0.1 - - [28/Jul/2015:16:44:50 -0700] "GET / HTTP/1.1" 303 - 0.0014
127.0.0.1 - - [28/Jul/2015:16:44:50 -0700] "GET /documentation HTTP/1.1" 200 3113 0.0024
```

## Generating Documentation
Comments are YARD compatible, but generated documentation is incomplete due to route extentions. 
This may be fixed in future versions of yard. YARD style tags are used and there is a .yardopts 
file for preferences. 

```
yardoc
```

## License

See [LICENSE](LICENSE).

## Support

This software is not meant to be run outside the context of the Zephir metadata management system. 
Tthe running service instance is supported as part of Zephir.
