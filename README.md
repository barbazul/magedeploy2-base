# MageDeploy2 Base

Magento2 Deployment Setup using Robo and Deployer. 
This is the base project you should base your deployments on.
It provides an configurable and customizable setup to create push deployments for Magento2.

## Getting Started

### Requirements

 * deployer/deployer
 * consolidation/robo
 * mwltr/magedeploy2
 * netz98/n98-deployer

### Prerequisites

MageDeploy2 requires deployer and robo to be available on your system.

Those Tools can be used globaly or added as a requirement to your local composer.json.

The path to those tools can be configured in the magedeploy2.php

### Installation

Create a new deployment setup

```
composer create-project mwltr/magedeploy2-base <dir>
```

Robo needs to be installed using composer, otherwise the usage of custom Tasks is not available.
See the Robo Documentation [Including Additional Tasks](http://robo.li/extending/#including-additional-tasks)

### Configure Deployment

After the Installation you have to edit the `magedeploy2.php` and the `deploy.php` file to suit your needs.
This tool assumes you have a git repository containing the magento composer.json. 
Furthermore your local build environment can clone said repository and download the Magento packages using composer.  

#### MageDeploy2 Configuration

To configure the MageDeploy2 use the following command:
```
./vendor/bin/robo config:init
```

It will guide you throught the most important configuration options. 
Don't worry you can edit the magedeploy2.php lateron.

Next, run 
```
./vendor/bin/robo validate
```
to validate your build environment is setup.

#### Setup local build environment

If you are done with the configuration in ``magedeploy2.php``, you can see if your build environment can be setup.
To do so run this command:
```
./vendor/bin/robo deploy:magento-setup develop
```
You can use a different branch depending on your git repository setup.

After the magento-setup has run successfully, you can now generate the assets by running the command:
```
./vendor/bin/robo deploy:artifacts-generate
```

After this command is complete you should see the packages beneath ``shop``.

At this point we are sure that the local build setup is working and we can now continue with releasing our project.

#### Deployer Configuration

To evaluate we will create a local deployment target.
To do so create run 
```
cp config/local.php.dist config/local.php
```
and set the config values according to your local deploy target.

Check the configuration in ``deploy.php`` and adjust it to your requirements.
The default configurations and tasks are defined in ``\N98\Deployer\Recipe\Magento2Recipe``.
You can also have a look at all the configurations available 
in the [Deployer Documentation](https://deployer.org/docs/configuration)

#### Setting up deploy directory tree

After you are done with setting the configuration, you can now initialize the directory tree of the deploy target run 
```
./vendor/bin/dep deploy:prepare local
```
This will create the required directories on your local deploy target.

#### Setting up deploy target (optional)
If you want to set up your deploy target as well you can use the command
```
./vendor/bin/dep server:setup local
```
It will make an initial deployment to push your code to the deploy target.

When this is done navigate to your local ``deploy_path`` and run the magento install command to setup the database.
This might look something like this:
```
cd <deploy_path>
php bin/magento setup:install --db-host=127.0.0.1 --db-name=magedeploy2_dev_test_1_server --db-user=root --admin-email=admin@mwltr.de 
--admin-firstname=Admin --admin-lastname=Admin --admin-password=admin123 --admin-user=admin 
--backend-frontname=admin --base-url=http://magedeploy2_dev --base-url-secure=https://magedeploy2_dev 
--currency=EUR --language=en_US --session-save=files --timezone=Europe/Berlin --use-rewrites=1
```

Now we have Magento database and configuration on our deploy target and are ready to continue with the final step.

#### Deploying the project

At this point, you have setup the build environment and target environment and can finally start with the actual deployment.
You can do so by running:
```
./vendor/bin/dep deploy local
```

Congrats you have successfully setup your deployment pipeline and run the first deployment!

## CONFIGURATION FILES

### magedeploy2.php

This is the config file to set all parameters required for the deployment in general.

The most common settings are to adjust on a project basis:

 - deploy/git_url (path to your git repository)
 - deploy/themes (the themes that are to be generated)
 - build/db (the database settings for the build environment)

A complete list of configuration options is provided further down.

#### env

| Key          | Description                 | Default                      |                        
| ------------ | --------------------------- | ---------------------------- |
| git_bin      | Path to git executable      | /usr/local/bin/git           |
| php_bin      | Path to php executable      | /usr/local/bin/php           |
| tar_bin      | Path to tar executable      | /usr/local/bin/gtar          |
| composer_bin | Path to composer executable | /usr/local/bin/composer.phar |
| deployer_bin | Path to deployer executable | /usr/local/bin/deployer.phar |

#### deploy

| Key           | Description                                                   | Default   |                        
| ------------- | ------------------------------------------------------------- | --------- |
| git_url       | Git-Url to your repository                                    |           |
| git_dir       | Sub-directory on build server to clone the repository to      | shop      |
| app_dir       | If you have your magento composer.json in another sub-dir     |           |
| themes        | An array of themes to compile                                 |           |
| assets        | List of assets to generate                                    |           |
| clean_dirs    | list of dirs to clean on each deployment                      |           |

#### build/db

Contains the database configuration being used to create and update a local magento setup during deployment.

### deploy.php

This file is for the actual deployment of your project to the server.
It has a basic setup that should work on most of the environments.
You can adjust this to your needs as well.
Leave out unwanted steps, add new ones, exchange existing ones, etc.

The `deploy.php` uses the `N98\Deploy\Recipe\Magento2Recipe` as base recipe and overwrites some of the default settings.

The server configuration is defined using the `config/*.php` files.

## COMMANDS

### config:init

Generate a new magedeploy2.php. Be aware: this will overwrite your existing configuration

### validate

Validates that 
- the bin config values are execuatble,
- the git repository is reachable

### deploy

Triggers the deployment with all it's stages

### deploy:magento-setup

Runs all tasks in the stage magento-setup. 
It will setup or update a local Magento instance by pulling the source-code from git, 
installing composer dependencies and installing or updating a local database.

### deploy:artifacts-generate

Runs the Magento ``di:compile`` and ``setup:static-content-deploy``commands to generate the assets.
It is using your configurartion from the ``magedeploy2.php``.

After generating those assets it will create packages, again according to your configuration.

### deploy:deploy

This command will invoke deployer to release your project and push the perpared artifacts to the server.

## Versioning

We use [SemVer](http://semver.org/) for versioning. 
For the versions available, see the [tags on this repository](https://github.com/mwr/magedeploy2-skeleton/tags). 

## Authors

* **Matthias Walter** - *Initial work* - [mwr](https://github.com/mwr)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
