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
See Zephir host readme for instructions on how to install and configure Zephir-API and required packages and environmentfiles. 
https://github.com/cdlib/htmm-env/blob/master/hosts/zephir/README.md

## Install gem dependencies with bundler

```
cd ~/ apps/zephir-api/
bundle install
```

## Setup database configuration file

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
