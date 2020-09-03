# Marketplace
myclimateservices.eu Marketplace Management Project. 

## Description
This describes the marketplace of myclimateservices.eu

Due to many reasons code is not hosted here but as private repositories on GitLab. Access to the repositories can be asked via Smart Cities Consulting
the repositorie is:
* marketplace.myclimateservices.eu -> https://gitlab.com/smart-cities-consulting/mcs-marketplace

## Implementation

The marketplace is a separate Drupal 8 website which can be hosted on any (virtual) server which hosts a LAMP system (Linux Apache Mysql PHP) with installed php package manager [composer](https://getcomposer.org/). It participates on the Single Sign On of profile.myclimateservices.eu. So the user can switch between CSIS and Marketplace without need to login on another page again.

Code is mainly managed by composer packages only custom/own code dedicated to marketplace is located in the repository. Custom/own Code which is most likley usable for other pages as well(profile, myclimateservices,events.myclimateservices.eu,...)  is managed using a own composer [repository at https://repository.myclimateservices.eu/](repository at https://repository.myclimateservices.eu). This repository is built with [satis](https://github.com/composer/satis) and fetches the code from the gitlab repositores of the custom drupal modules. 

Syncronisation of needed data of users and organisations from profile.myclimateservices.eu is done by using the REST endpoints on profile created for marketplace. The syncronisation uses drupals built in module migrate to call the endpoints, which expose the useres and organistions created and changed in the last 5 minutes. The neccessary migrations are run by a shell script which is called by cron every 5 minutes and runs the drush commands for migrations

```bash
#!/bin/bash
cd /directory/of/marketplace

vendor/bin/drush mrs sync_organisations
vendor/bin/drush mim sync_organisations --update
vendor/bin/drush mrs sync_users_from_profile
vendor/bin/drush mim sync_users_from_profile --update
vendor/bin/drush mrs link_users_to_cas
vendor/bin/drush mim link_users_to_cas --update

```

## Deployment

### Installing the code base

```bash
cd /path/to/website
git clone https://gitlab.com/smart-cities-consulting/mcs-marketplace.git ./
composer install
composer drupal:scaffold

```

### Install Drupal

Create a database for the System:

```bash
mysql -u root -p
CREATE DATABASE <databasename>;
GRANT ALL PRIVILEGES on <databasename>.* to "<dbuser>"@"localhost" IDENTIFIED BY "<password>";
```
Open a web browser and visit the url of the page. 
Follow the install wizard of Drupal 8 and enter your database credentials when asked

### import Drupals configuration

```bash
cd /path/to/website
vim web/sites/default/setting.php  -> $config_directories['sync'] = '../config/sync';
vendor/bin/drush cim 
vendor/bin/drush cr

```

Ćreate the script and cron job to syncronise data from profile.myclimateservces.eu

## Syncronisation between Dev and Prod

There is no coentent which has to be developed on a development environment and the production systen. So only code and configuration has to be transfered from dev to prod.  On the Dev system do from the directory with the composer.json:

```bash
vendor/bin/drush cr
vendor/bin/drush cex
git add --all
git commit
git pull
git push

```
This exports first the changes in configuration and updates then the repository with the changes
Then on the production system:

```bash
git pull
composer install
vendor/bin/drush updb
vendor/bin/drush cim
vendor/bin/drush cr

```
This upda. tes the codebase the composer.json and composer.lock files Then composer is told to install new packages or update old ones if the where updated on dev. Afterwards drupals databes is updated if the code changes in alreasy activated modules have an effect on the database. The configuration is imported and the cache are flushed to be sure all changes are applied for the user.

## Update

Drupal good practices tell to update first the dev system and check if nothing will break. In the directory with the composer.json file:
```bash
composer update --with-dependencies
vendor/bin/drush updb
vendor/bin/drush cr
```
After that check if 
* composer applied pathes 
* the page is not broken

If everything on the dev system is OK sync dev to prod as described above. Sometimes the update process will require more steps like deleting directories or update a single module in two steps. In that case you have sync to prod after ever single step or replay the update process on prod instead the to update only with this procedure
