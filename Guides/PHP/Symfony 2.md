#Deploying Symfony 2 to cloudControl

In order to deploy a [Symfony2](http://symfony.com/) Application we need to take several steps.

## Dependency Management and deploxment Composer
Symfony2 Application come by default with [Composer](https://getcomposer.org/) as dependency management. Within a composer.json you define the dependencies of your project.
Composer itself is already included in the cloudcontrol [php buildback](https://github.com/cloudControl/buildpack-php)

In order to deploy our Symfony2 application we are utilizing composer hooks to execute commands. Symfony2 Standard edition already contains the default commands for our deployment.

### Preventing deployment hooks from running in the wrong environment
One of the main issues with deploying with composer is that composer is not aware of the environment (e.g. prod / dev / env). So all those hooks are executed in the dev environment. 
Therefore we need to prevent default execution of the hooks.

Create a envrc file under the .buildpack folder in your project root:

.buildpack/envrc
	export COMPOSER_INSTALL_ARGS="--no-scripts --optimize-autoloader"

### Running deployment hooks with prepared environment
In order to run the necesasry commands in prod environemt we need to define a bash script.
Create a warmup.sh in your project root:

warmup.sh:

	php code/vendor/sensio/distribution-bundle/Sensio/Bundle/DistributionBundle/Resources/bin/build_bootstrap.php;
	php installer;
	php composer.phar run-script post-install-cmd -d code;
	php code/app/console --env=prod --no-debug cache:clear --no-warmup;
	php code/app/console --env=prod --no-debug cache:warmup;
	php code/app/console --env=prod doctrine:ensure-production-settings;
	
The execution of this script is triggered by a custom [Procfile](https://www.cloudcontrol.com/dev-center/Platform%20Documentation#version-control--images). 

Procfile:

	web: bash code/warmup.sh; bash boot.sh 
	
### Addon credentials for our application
In order to provide our Application with addon credentials we are utilizing composer with its [Incenteev\ParameterHandler](https://github.com/Incenteev/ParameterHandler).

composer.json:

	    "extra": {
        "symfony-app-dir": "app",
        "symfony-web-dir": "web",
        "incenteev-parameters": {
            "file": "app/config/parameters.yml",
            "dist-file": "app/config//parameters.yml.dist",
            "env-map": {
                "database_name": "MYSQLS_DATABASE",
                "database_user": "MYSQLS_USERNAME",
                "database_password":"MYSQLS_PASSWORD",
                "database_host" : "MYSQLS_HOSTNAME"
            }
        }
    }
	
Here we map our addon credentials from the environment to our parameters defined in our symfony2 config.yml.

## Logs
@todo: describe a good solution for logging