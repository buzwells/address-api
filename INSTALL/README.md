# Requirements

# Create a Virtual Box with Address API

The following software will be installed

* Ubuntu 14.04.3 LTS - trusty
* PHP 5.5.9
* Apache/2.4.7
* Postgresql 9.3.11 - server encoding UTF8, installed extensions

  * fuzzystrmatch     1.0     - determine similarities and distance between strings
  * ogr_fdw           1.0     - foreign-data wrapper for GIS data access - https://github.com/pramsey/pgsql-ogr-fdw
  * plpgsql           1.0     - PL/pgSQL procedural language
  * postgis           2.1.2   - PostGIS geometry, geography, and raster spatial types and functions
  * postgis_topology  2.1.2   - PostGIS topology spatial types and functions
  * postgres_fdw      1.0     - foreign-data wrapper for remote PostgreSQL servers

* PostGIS 2.1.2 r12389
  * GDAL 1.11.2, released 2015/02/10

## Requirements

* Make certain ssh can be executed and it must be in you path  
* [Virtual Box](https://www.virtualbox.org/)
* [Vagrant](https://www.vagrantup.com/)


## Clone repository

````
    git clone git@github.com:codeforkansascity/address-api.git
    cd address-api
````

## Setup data to be restored

You need to create a directory called `dumps` and copy the dump files to them

````
    mkdir dumps
````

Copy he dumps from https://drive.google.com/drive/u/0/folders/0B1F5BJsDsPCXb2NYSmxCT09TX1k
to the dumps directory and unzip them

````
   cd dumps

   gunzip address_api-20160220-0548.dump.gz
   gunzip code4kc-20160220-0548.dump.gz

   cd ..
````

## Create image


````
    vagrant up
````


## Login


````
    vagrant ssh
````

# Setup postgres

## Login as postgres
````
sudo su - postgres
````


## Restore databases


````
   cd /var/www/dumps

   pg_restore -C -d address_api address_api-20160220-0548.dump
   pg_restore -C -d code4kc code4kc-20160220-0548.dump
````

You will get several errors but you can ignor them

## Fix ownerships

Run the "fix_ownerships.psql" script, as shown below. Tap the "q" key to continue when execution pauses.

````
cd /var/www/data/scripts
psql -f fix_ownerships.psql
exit
````

## Restart postgres

````
sudo service postgresql stop
sudo service postgresql start
````

# Setup host environment

On your computer add the following to /etc/hosts


````
192.168.33.11 dev-api.codeforkc.local
````


You should now be able to browse to the following

````
http://dev-api.codeforkc.local/address-attributes/V0/210%20W%2019TH%20TER%20FL%201%2C?city=Kansas%20City&state=mo
````

And see

````
{"code":200,"status":"success","message":"","data":{"id":200567,"single_line_address":"210 W 19TH TER FL 1, KANSAS CITY...
````

You should also be able to navigate to the api documentation:

````
http://dev-api.codeforkc.local/api-docs
````


You should have access to OGR Foreign Data Wrapper see https://github.com/pramsey/pgsql-ogr-fdw
### Notes


We need to figure out how if we need sfcgal and address_standardizer and if so how to install them.

````
CREATE EXTENSION postgis_sfcgal;
CREATE EXTENSION address_standardizer;
````

# Installing `gh-pages` and Jekyll


Do this above the `address-api` directory, so both are at the same level.

1 .Create clone of `address-api` and switch to the `gh-pages` branch

````
git clone git@github.com:zmon/address-api.git address-api-gh-pages
cd address-api-gh-pages
git remote add upstream git@github.com:codeforkansascity/address-api.git
git checkout gh-pages
````

2. Make it so your local git does not include CNAME by editing .git/info/exclude and adding the line

````
CNAME
````

2. Contect `address-api-gh-pages` to your virual box

Edit `Vagrantfile` in the `address-api` directory, and add the following line at about line 42.

````
  config.vm.synced_folder "../address-api-gh-pages", "/var/www-gh-pages"
````

3. Start or restart your Vagrant box from the `address-api` directory.

````
vagrant reload
````

4. Log into the Vagrant box

````
vagrant ssh
````

5. Install Ruby 2.2.3


````
sudo su -
rvm install ruby-2.2.3
rvm use ruby-2.2.3 --default
gem install jekyll

````

May need to do

````
sudo apt-get install ruby-dev
````

Start Jekyll

````
cd /var/www-gh-pages
jekyll serve --host 0.0.0.0
````

Now go view the site at http://192.168.33.11:4000/
