# MVP 3 - Install instructions

## Notes

_As of March 2020_, ISLE 8 prototype using only wodby images e.g. Drupal, PHP, Solr and Mariadb

* The `composer.json` used to install is from the following source(s)
  * _currently_ https://github.com/Born-Digital-US/drupal-project/tree/isle8-dev
    * which is a fork of https://github.com/Islandora/drupal-project
  * on the `isle8-dev` branch there is a new `composer.json` file which was the resulting file from running https://github.com/Islandora-Devops/islandora-playbook
  * Ultimately https://github.com/Islandora/drupal-project will become the canonical source.

## Install instructions

Launch a terminal and follow these steps below:

* On your local, add the local domain/site to `/etc/hosts`
  * `sudo nano /etc/hosts`
  * add `127.0.0.1   idcp.localhost` as a seperate line underneath `127.0.0.1       localhost`
  * save and exit the hosts file

* Within the terminal, navigate to a directory of your choice that you can start working

* Clone the ISLE 8 project (isle-dc)
* `git clone https://github.com/Islandora-Devops/isle-dc.git`

* `cd isle-dc`

* Generate public/private key pair for JWTs (only has to be done once)
  * `(cd scripts; ./generate_keys.sh)`

* Pull down Docker images
  * `docker-compose pull`

* Start up the Docker containers
  * `docker-compose up -d`

* Generate public/private key pair for JWTs
  * `docker exec -it isle_dc_proto_solr bash -c "solr create_core -c islandora"`

* Create a new Solr core called "islandora"
  * `docker exec -it isle_dc_proto_solr bash -c "solr create_core -c islandora"`

* Run the Drupal site installation script
  * `docker exec -it isle_dc_proto_php bash -c "sh /scripts/islandora/install-islandora.sh"`
  * This script will take at least 5-10 mins depending on the speed of your internet connection and/or local environment.

* Access site at: http://idcp.localhost

* Test Houdini with by running `identify` on the Islandora logo:
  * `curl -H "Authorization: Bearer islandora" -H "Apix-Ldp-Resource: https://islandora.ca/sites/default/files/Islandora.png" idcp.localhost:8000/identify`

* The directory `/var/www/html` is bind mounted in both the Apache and PHP services / containers to the local directory `isle-dc/codebase`. This directory is in the .gitignore file to ignore the contents of this data directory.

* To shut down the containers but persist data
  * `docker-compose down`

* To **destroy** containers and data including the Drupal site and all content. This is essential when re-testing or re-installing.
  * `docker-compose down -v`
  * `sudo rm -rf codebase`

---

### Database settings

Change settings in the `php.env` and/or `.env` files

* MySQL root password: `root_pw`
* Database name: `drupal`
* Database user: `drupal_user`
* Database user password: `drupal_user_pw`
* Database host: `mariadb`
* Database Port number: `3306`
* Table name prefix: `left empty / blank`
* Database type: `MySQL, MariaDB, Percona Server, or equivalent`

### Drupal settings

Change settings in the `php.env` and/or `.env` files

* Site url: http://idcp.localhost
* Drupal site name: `ISLE 8 Local`
* Drupal user: `islandora`
* Drupal user password: `islandora`
* Drupal site email address: `admin@example.com`
* Drupal user email address: `islandora@example.com`
* Drupal 8 installation profile: `Standard`
* Drupal Language: English `en`
* Drupal Locale: `US`

### Ports / services / dashboards

* ActiveMQ
  * Port `8161`
  * http://idcp.localhost:8161

* Solr
  * Port: **To do:** _expose port for admin access and document_
  * http://idcp.localhost

### Followup To DOs

**TO DO** - Test if Solr is indexing? (MVP 2)
**TO DO** - Sample items with metadata (MVP 2)
**TO DO** - What is content / search setup aka block? (MVP 2)

---

# MVP 2 - Alpaca connectors and activemq

* Aaron Birkland's https://github.com/Islandora-Devops/isle-dc/pull/7

Adds a single Karaf instance containing all connectors, config to run them, and an activemq instance

## To Test

* `docker-compose up -d`
  * This should pull in all images, if not do a `docker-compose pull` first
* Go to http://localhost:8161, which is the ActiveMQ admin console.
  * Click on "Manage ActiveMQ broker"
  * Enter in username `admin`, pass `admin`
* Click on the `queues` tab. You should see all the queues the connectors listen to.
* Now, pick a service (maybe fits) and remember it.
  * Shut everything down `docker-compose down -v`
* Within the `.env` file, comment out the env vars for that service. This will disable it.
* Start up again `docker-compose up -d`
* Log into ActiveMQ.
  * Look at the queues, and observe the one that corresponds to your service doesn't exist. 
  * This proves you can shut them off at will

There isn't much more to test, since services aren't connected.

This PR just verifies that camel routes start successfully, connect to the messaging bus and eagerly await messages that never come.

---

## MVP3 Instructions

* To install run Fedora (MVP3), the following:
  * the `fedora` sql file has been mounted into mysql `- ./scripts/fcrepo/sqlfiles:/docker-entrypoint-initdb.d` (_this should automatically create and install all fcrepo db users and perms_)
  * the `gemini` sql file has been mounted into mysql `- ./scripts/fcrepo/sqlfiles:/docker-entrypoint-initdb.d` (_this should automatically create and install all gemnni db users and perms_)
  * This will only run if starting up for the first time, if one attempts to add fcrepo to a previously existing MVP2 it will fail due to a limitation by `wodby`
  * Within a terminal and within the project directory run the following:
    * `bash scripts/fcrepo/generate_syn_key.sh`

* Blazegraph namespace setup required
 * `docker exec -it isle_dc_proto_blazegraph bash /scripts/install_islandora_namespace.sh`
  * If this worked correctly, Blazegraph should respond with `"CREATED: islandora"` & with some XML letting us it added the 2 entries from inference.nt to the namespace.
