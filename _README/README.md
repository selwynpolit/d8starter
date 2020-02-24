D8Starter - Drupal 8 starter project
------------------------------------

##Local setup
1. clone repo
1. edit .ddev/config.yaml: change name: d8starter to the project name
1. ddev start
1. code and be happy



## DDEV usage

### Everyday ddev commands
`ddev start` - start containers

`ddev stop` - stop containers, delete them and **keep db intact**

`ddev restart` - rebuild containers if changing configuration of ddev

`ddev ssh` - ssh into container

`ddev sequelpro` - fire up sql pro already connected!

`ddev composer <blah>` - run composer in container e.g. `composer install`

`ddev describe` - show the sql connection ports etc. for connecting sequel pro

`ddev list` - list sites and urls

### More ddev commands

`ddev` - list all commands

`ddev <command> --help` - shows help about a command e.g. `ddev list --help`

`ddev import-db --src=dbdev.sql.gz` - import gzipped db file from host

`ddev export-db -f dbdump2.sql.gz` - export db file to host file dbdump1.sql.gz

`ddev rm --unlist` - remove the site from the ddev listing

`ddev pull` - special command to pull db from pantheon environment.  Please note. this will pull from the
most recent backup, not the current database file in use!!! So backup Pantheon first with `terminus backup:create txglobal.[env]`

### drush
Install drush using instructions at https://docs.drush.org/en/master/install/
 Many drush commands work as long as you are in the directory
e.g.

* drush status
* drush cr
* drush en <module>
* drush uli

when it doesn't just use `ddev exec drush <drush command>`  e.g.

* ddev exec drush uli
* ddev composer require drupal/admin_toolbar
* ddev exec drush sql-dump >dbdump2.sql
* ddev exec drush updb
* ddev exec drush cim sync
* ddev exec drush cex sync
* ddev exec drush cget system.site uuid


### Terminus

`terminus site:info` - get info about the site

`terminus drush cr` - clear cache on dev environment

`terminus drush cim sync -y`

`terminus drush updb`

Note. these commands will apply to the txglobal.dev environment.  To apply them to a different command, you have to specify site.test or site.live use terminus command:subcommand <site>.<env>

`terminus env:cc txglobal.dev`  - clear caches for dev enviro

`terminus env:cc txglobal.test`  - clear caches for test enviro

`terminus env:cc txglobal.live`  - clear caches for live enviro


### Login to Pantheon sites

Dev, test and live.
terminus drush txglobal.dev uli
terminus drush txglobal.test uli
terminus drush txglobal.live uli


More at https://pantheon.io/docs/terminus/commands/


See https://ddev.readthedocs.io/en/stable/users/providers/pantheon/ for configuring your local to use pantheon as a provider.
### Drush aliases for Pantheon
One notable command `terminus aliases` is interesting.  It writes the aliases to ~/.drush/pantheon.aliases.drushrc.php.

 `drush cc drush` - to clear drush cache

 `drush sa | grep inside`
 @pantheon.txglobal.dev
 @pantheon.txglobal.live
 @pantheon.txglobal.test

You can now issue drush commands directly against the Pantheon environments using:
 `drush @txglobal.dev st`
 `drush @txglobal.test st`
 `drush @txglobal.live st`


### Composer usage
1. composer require drupal/admin_toolbar
1. composer install -o --no-dev --no-interaction
Note. if you don't specify --no-dev, you will risk checking in development vendor software.
Also if you don't have php 7.3 on your mac you could rather use `ddev composer ..` to run composer in the container.

Note. if you've run composer to install modules locally, when you go to run composer --no-dev, you will see:
- Removing drupal/coder (8.3.1)
  [RuntimeException]
  Source directory /Users/selwyn/Sites/txglobal/vendor/drupal/coder has uncommitted changes.
You will need to `rm -fr vendor/drupal/coder/` and then re-run it
You may also need to `rm -fr vendor/behat/mink`




### To Enable/disable Xdebug in ddev
edit .ddev/config.yaml
change
xdebug_enabled: false
to
xdebug_enabled: true
$ ddev restart

Many people prefer to use:

ddev exec enable_xdebug
and 
ddev exec disable_xdebug

instead of configuring this, because xdebug has a significant performance impact.


### Twig debugging on local with ddev
* You need a sites/default/settings.local.php (a sample is provided in the _README folder)
with the following line:
```
/**
 * Enable local development services.
 */
$settings['container_yamls'][] = DRUPAL_ROOT . '/sites/development.services.yml';
```
* You need a file: sites/development.services.yml (not sites/default/development.services.yml curiously.)
You can either copy parts of this file from sites/default/default.services.yml or use the sample file that is provided
 in the _README folder

* After clearing caches with `drush cr` you should see twig debug info

* To disable twig debugging, edit this line in sites/development.services.yml so it reads:
    debug: false


### Terminus installation (Pantheon utility)
Following the steps from https://pantheon.io/docs/terminus/install/
1.First, create a Machine Token from User Dashboard > Account > Machine Tokens.
1.curl -O https://raw.githubusercontent.com/pantheon-systems/terminus-installer/master/builds/installer.phar && php installer.phar install
1.terminus auth:login --machine-token=‹machine-token›
Note. When you create a machine token on Pantheon site be sure to store a copy for later use with ddev below.


### Pantheon SSH key
Given: you can login to pantheon at pantheon.io
Follow the steps at https://pantheon.io/docs/ssh-keys to generate a key or use an existing key


### DDEV step-by-step installation for new site

Given: you have permissions on Pantheon, have setup an SSH key at Pantheon.io and have Terminus installed
1. Install docker-ce from https://hub.docker.com/editions/community/docker-ce-desktop-mac
1. brew tap drud/ddev && brew install ddev (to install ddev)
1. mkcert -install  (more ddev install stuff)
1. Navigate to ~/Sites
1. Retrieve the clone command from pantheon.io - click the yellow "clone with Git" button
1. git clone ssh://codeserver.dev.e0bd7728-f295-423b-8e1a-0a31665b90e8@codeserver.dev.e0bd7728-f295-423b-8e1a-0a31665b90e8.drush.in:2222/~/repository.git txglobal
1. cd txglobal
1. terminus auth:login --email spolit@mightycitizen.com
1. ddev auth pantheon <machine-token>
1. ddev config pantheon (press enter on all the defaults as long as they look sane)
1. ddev start - fire up the container
1. ddev pull - load the db from dev (backup Pantheon first if you want a fresh database)
1. Navigate to https://txglobal.ddev.local
1. drush uli -l https://txglobal.ddev.local
1. ddev stop - to shut down the site

Note. if you need to specify the port for your local URL i.e. 80 and 443
create a .ddev/config.local.yaml with the following contents:

```
# my local ddev
# See https://ddev.readthedocs.io/en/stable/users/extend/config_yaml/ for poss values

router_http_port: "80"
router_https_port: "443"
```


### Configuration workflow

From: https://pantheon.io/docs/drupal-8-configuration-management/

To push config changes from local environment to dev (on Pantheon)

1. First check if there are configuration changes (in your local database) from dev that you need to import with `drush cst`
1. If there are changes, you should import them to your local db with `drush cim -y`
1.	Make local changes.
1.  Export config with `drush cex` - This will write config files to /config
1.	Commit them to git
1.	Push repo to Pantheon
1.	terminus drush cr (terminus drush cr or terminus env:cc txglobal.dev)
1.	terminus drush cim -y
1.	terminus drush updb if necessary
1.	done!

To make changes on pantheon dev
1.	Set pantheon into SFTP mode (not git)
2.	Make changes on pantheon dev environment (in Drupal)
3.	terminus drush d8-pantheon.dev -- cex -y
4.	terminus env:commit d8-pantheon.dev --message="Export config for event content type changes"
5.	Now you can pull the changes down to local and drush cim sync them into the db.
(Substitute .dev for .live if you want to grab the config from the production site)


### Configuration Protection

#### Config Readonly
The *config_readonly* module limits users from making changes to the config from the Drupal U/I.
In settings.php, a whitelist pattern is specified like that listed below:
```
$settings['config_readonly_whitelist_patterns'] = [
  'system.menu.main*',
  'system.menu.utility*',
  'system.menu.footer*',
  'webform.webform.*',
  'system.site',
  'webform*',
  'system.files',
  'migrate*',
  'webform.*',
  'captcha.*',
  'recaptcha.*',
  'metatag.*',
  'simple_sitemap.*',
  'danamod.header_footer_settings',
];
```

Specify `$settings['config_readonly'] = false;` in settings.local.php to disable this functionality for local development.

The *config_ignore* module is used to bypass loading (think drush cim) configuration from the config directory.
You can specify the configuration items to never load at /admin/config/development/configuration/ignore

#### Config Ignore

If you go to `/admin/config/development/configuration/ignore`
you will see a fairly simple interface.

Add the name of the configuration that you want to ignore upon import.
(e.g. `system.site` to ignore site name, slogan and email site email address.)
Click the "Save configuration" button and you are good to go.

Do not ignore the `core.extension` configuration as it will prevent you
from enabling new modules with a config import. Use the `config_split` module
for environment specific modules.

If you need to bypass Config Ignore you can update/create a single configuration
by using the "Single import" feature found at
`admin/config/development/configuration/single/import`.

To deactivate `config_ignore`, include
`$settings['config_ignore_deactivate'] = TRUE;`

#### Config Split

Add the following lines to the settings.local.php in web/sites/default
```
// Config split
// The following line represents the dev and local environment.
$config['config_split.config_split.dev']['status'] = TRUE;
// The following line represents the test and prod environment.
//$config['config_split.config_split.test']['status'] = TRUE;
```

#### To add a module to dev only (local and Pantheon dev)

1. in settings.local.php, ensure this line exists: `$config['config_split.config_split.dev']['status'] = TRUE;`
1. composer require `<said module>`
1. drush en <said module>
1. drush cr
1. drush cex to export config
1. add config to git the usual way
1. push to Pantheon


#### To add a module to test (Pantheon test and prod)

1. in settings.local.php, comment out the line: `$config['config_split.config_split.dev']['status'] = TRUE;`
1. drush cr
1. composer require `<said module>`
1. drush en `<said module>`
1. drush cex to export config
1. add config to git the usual way
1. uncomment the line you commented out above



### Developer workflow for deployment (to be compatible with Pantheon)

```
git checkout master
git pull  (run a backup on Pantheon if you want a really recent database)
git checkout -b feature/blah
work
drush cst (make sure you have committed any config changes)
git pull origin master
drush cst (did someone else make config changes? - be sure to drush cim them)
git checkout master
git merge feature/blah
git push
```

Note: Running `git push` will automatically deploy to the Pantheon dev server

To deploy to test and live (and clear caches) use:

`terminus env:deploy txglobal.test --note="Deployed via terminus" --cc`

`terminus env:deploy txglobal.live --note="Deployed via terminus" --cc`

Clear Drupal Caches on Pantheon servers

```
terminus drush txglobal.dev cr
terminus drush txglobal.test cr
terminus drush txglobal.live cr
```


### Building the front-end for this project

Confirm you are using node version 10

`node -v`

You should see: v10.16.3

```
pushd ~/Sites/txglobal/web/themes/custom/txglobal/foundation
nvm use 10
npm install
npm start
popd
```

The `npm start` to open a browser and allow you to watch changes

if it fails, try
```
rm -fr node_modules
npm start
```

if it still fails, build the magical Kristine globe below

```
pushd src/assets/js/globe/
nvm use 10
npm install
popd
```

Note. Changes to JS should be made in web/themes/custom/txglobal/foundation/src/assets/js/app.js followed by rebuilding the front end (above)


### New version of ddev
You will need to update your ddev files and check them into git

shut down your sites with `ddev poweroff`
update ddev executable with `brew upgrade ddev`
in the project directory, `ddev config pantheon`
Press enter through all the defaults
review changes and check in .ddev directory


## Helpful scripts to make a dev's life cheerier

### Helpful backup script
I keep this in ~/bin.ddevbackup.sh.
It keeps a rotating set of backup files dbdump1.sql.gz and dbdump2.sql.gz.
Run this file to make a copy of the most recent backup and to create a new backup.  I use it any time I am making big changes to the database and I think I might need to be able to roll back.
Before running it, make the 2 backup files with:
```
cd web
ddev export-db -f dbdump1.sql.gz
ddev export-db -f dbdump2.sql.gz
```

```
#!/bin/bash
set -e
set -x

FILE=./dbdump2.sql.gz
if [ ! -f "$FILE" ]; then
    echo "Bailing, no dbdump2.sql found here"
    exit 1
fi

    ls -al db*

    echo "$FILE exists, backing up.."

    mv dbdump2.sql.gz dbdump1.sql.gz
    ddev export-db -f dbdump2.sql.gz
    ls -al db*
```

### Helpful bup.sh script
I keep this one in ~/Sites/txglobal/web
It makes Pantheon do a backup, retrieves that backup and loads it into the ddev database container.

Run this any time you need a fresh database from Pantheon dev site
Note. this does depend on your using the ddevbackup.sh helper script also as it checks for the existence of a dbdump2.sql.gz file.

Note also that you will have to do a drush uli to log into your local environment as the database has changed and your tokens are all out of date.

```
#!/bin/bash
#set -e
set -x

echo "Backing up txglobal site on pantheon"
terminus backup:create -- txglobal.dev

FILE=./dbdump2.sql.gz
if [ ! -f "$FILE" ]; then
    echo "Bailing, no dbdump2.sql found here.  Do ddevbackup.sh first"
    exit 1
fi


echo "Displaying local backups"
ls -al ~/Sites/txglobal/web/db*

read -r -p "Loading pantheon db into ddev. Are you sure? [y/N] " response
if [[ "$response" =~ ^([yY][eE][sS]|[yY])+$ ]]
then
    ddev pull
else
    echo "Skipping pull.  No data imported"
fi

```
Dann

## Troubleshooting

### Failed to get project
If you try to start the ddev environment (e.g. ddev start) and you get:

Failed to get project(s): could not find a pantheon site named txglobal

You will need to re-authorize ddev & pantheon with your machine id:

`ddev auth-pantheon <machine-id>`

### Containers won't start

ddev stop --remove-data --omit-snapshot. - if db corrupt and containers won’t start.  Follow this up with ddev start.  You may also have to restart docker.


### You are not logged in error when issuing terminus command
If you try a terminus command and see:
In Authorizer.php line 37:

  You are not logged in. Run `auth:login` to authenticate or `help auth:login` for more info.

You need to re-authenticate with Pantheon.
$ `terminus auth:login --email=spolit@mightycitizen.com`
 [notice] Logging in via machine token.

### ddev pull returns "Pull failed: could not find a pantheon site named"
sometimes it seems ddev can get confused and stubbornly won't download the db backup from Panthon

To fix this simply run
```
ddev config pantheon
```
Follow the prompts below:
Project name (txglobal): <enter>
Docroot Location (web): <enter>
Project Type [php, drupal6, drupal7, drupal8, wordpress, typo3, backdrop] (drupal8): <enter>
Configure import environment:
	- test
	- live
	- dev
Type the name to select an environment to pull from (dev): <enter>

```
ddev restart
```
