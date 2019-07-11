# ivonet/mysql Docker image

A Made specific image for MySQL based on the standard mysql image.

This image has to be build on your local machine to make the 
development environment work.

This image is tuned to monitor a TEST_DIR folder file up and running.
if you put sql files into this folder they will be executed on the database
and when done removed.

## How to build

* Run `build.sh` from this folder

## What it does

* Create a mysql image
* Add a custom entrypoint
* Configures mysql to monitor for testdata
* Create an image tagged as `ivonet/mysql`
* Push the image to docker hub


## Why

My version of the MySql image exposes a `/testdata` folder and that folder is monitored for changes.
When changes detected it will execute it on the running database.
You can put testdata `*.sql` files there. Note that the files will also be removed from that folder after it has been consumed against the db. Only `*.sql` files will be cunsumed. all other files will be ignored. A log file in that folder will give feedback on what happened.

## A docker compose example

The `docker-compose.yml` file below exposes two native volumes:

./volumes/setup: Put your setup *.sql files here. e.g. CREATE statements 
./volumes/testdata: copy your testdata *.sql files here. These files will be 'consumed' into the database. 

It will also enable a phpmyadmin for further db control...

```text
version: '2'

services:
  mysql:
    image: ivonet/mysql
    volumes:
      - ./volumes/setup:/docker-entrypoint-initdb.d
      - ./volumes/testdata/mysql:/testdata
      - mysql-data:/var/lib/mysql
    ports:
      - "3306:3306"
    environment:
        - MYSQL_ROOT_PASSWORD=secret

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:4.6.4-1
    ports:
      - "8888:80"
    links:
      - mysql:mysql
    environment:
      - MYSQL_USERNAME=root
      - MYSQL_ROOT_PASSWORD=secret
      - PMA_HOST=mysql
    depends_on:
      - mysql
      
volumes:
  mysql-data:
    driver: local

```
