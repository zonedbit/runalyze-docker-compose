# Installation

## Precondition
Docker and docker-compose must be installed before it is possible to use this project to setup Runalyze

The details of the installation might be variate from the used operation system, there should be plenty of instruction.

## Bugfix runalyze
There are two possible approaches to fix runalyze.

The first approach is to patch the [runalyze source code](#Patch-the-runalyze-code). It is just a minor change, but a bit time consuming. However, in the case of a re-installation it is possible to skip the bug-fixing part.

As an alternative to patch the source code, it is possible to add the missing column [after the installation](#Bugfix-after-installation) to the table.

The next two section will give guidance to both approaches.

### Patch the runalyze code
Download the latest version of runalyze:
```bash
cd web-app\version

wget -O runalyze_unfixed.tar.gz https://github.com/Runalyze/Runalyze/releases/download/v4.3.0/runalyze-v4.3.0.tar.gz
```

Open the file *structure.sql*
```bash
tar -xzvf runalyze_unfixed.tar.gz

vi runalyze/inc/install/structure.sql
```

After line 350 add the line:
```sql
`is_power_calculated` TINYINT(1) DEFAULT NULL,
```

Change the ownership of the folder *runalyze*
```bash
sudo chown www-data:www-data -R runalyze
```

Create tarball of the fixed version
```bash
tar -zcvf runalyze.tar.gz runalyze
```

The folder *runalyze* and the file *runalyze_unfixed.tar.gz* is no longer needed.
```bash
sudo rm -rf runalyze runalyze_unfixed.tar.gz
```

### Bugfix after installation
``` bash
docker exec -it runalyze-docker_db_1 /bin/bash

mysql -u runalyze -p runalyze

ALTER TABLE `runalyzetraining` ADD `is_power_calculated` TINYINT(1) DEFAULT NULL AFTER `power`;
```

## Install runalyze

# API Keys
By providing API keys it is possible to fetch information from a third party API such as weather data.
Therefore it is necessary to provide API keys for the services, which should be used.

The API keys must be configured in the file *web-app/data/config.yml*.
To avoid conflicts by updating this project, it is recommended to create your custom configuration files.
This can be achieved be copy the config.yml file and providing the new name of the file as
[environment variable](Customize_configuration_-_Environment_files).

# Create database tables via browser
Start the docker containers with docker-compose:
``` bash
# Start the container in the foreground
docker-compose up

# Start the container in the background
docker-compose up -d

# Stop the container
docker-compose down
```
If the *webapp* container does not start correctly, use `docker-compose down` and then `docker-compose up`

### Start the setup process of runalyze
- Open the URL *http://127.0.0.1:8000/install/start* in your browser
- Follow the instruction on the screen to setup runalyze. After the successful setup, a new user account must be created.
- Congratulation, you should be able to login to Runalyze via *http://127.0.0.1:8000* and upload your first activity. *Enjoy Running :-)*

## Customize configuration - Environment files
It might be helpful to use a customize configuration, e.g. to use API-Keys to retrieve the weather data of your activity.

To avoid conflicts with the provided default configuration -e.g. overwriting files by a git pull- you can create a docker-compose
environment file and provide the values, see example below

``` bash
touch .env

# the name of the docker volume for the database; Default value: *default_config.yml*
echo "RUNALYZE_DATA_VOLUME=runalyze-docker_runalyze_data" >> .env

# the name of the custom configuration file; Default value: *default_runalyze_data_volume*
echo "RUNALYZE_WEB_APP_CONFIG=config.yml" >> .env

# Cope the configuration file
cp web-app/data/config/default_config.yml web-app/data/config/config.yml

# Change the values or provide the API Keys
vi web-app/data/config/config.yml

```