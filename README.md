## Overview
This is a Docker image / container for running SocialEngine on NGINX with PHP-FPM.
This repository does **not** contain any SocialEngine components. Rather, it is a container configured to run SocialEngine and support DevOps workflows. It is designed to work with a SocialEngine instance (code and database) stored in Github. Note that because SocialEngine is a commercial product, it should not be stored in a public repository. Github supports private repositories for individuals and organizations, though you need a paid account to take advantage of them. You can test this without a Github repository.
This image supports modern DevOps workflows around SE, using a Docker container for the application environment, and loading your SocialEngine site code and database from Github. Using Docker Compose, you can run a fully self-contained development environment on your laptop, push changes to source control and have them instantly up and running on a container-based server. I've currently got this running on the Google Cloud Platform Compute Engine, using a VPS that is initialized with the social-nginx image.
This initial version is a work in progress. I currently don't have SSL set up (though that shouldn't be difficult). I'm currently in early development on a SocialEngine site so I'll be updating this as I go. I would welcome others to fork this repository, and contribute your improvements through pull requests. If you experience problems, you may also open an issue on the Github project page.
Next steps:
* Implement SSL
* Optimize and clean up NGINX & PHP config. Perhaps break PHP-FPM into a separate container?
* Optimize & harden directory permissions for dev & production.
* Incorporate other PHP modules as needed
* Test on GCP Kubernetes Engine
* Implement a test harness for automated testing
* Implement a database migration strategy, supporting continuous delivery all the way into production.
* Automate CD using Jenkins or a cloud DevOps pipeline
* Test on AWS and Azure
* Update social-nginx docs
This will result in full Continuous Integration and Delivery of even large SocialEngine installations, while enabling developers to easily work on isolated feature branches.
This leads me to an important suggestion for the SocialEngine team. SocialEngine is licensed in an old-school "paid script" model where everybody hacks on one instance of the script running on some VPS. To support this old model SE allows only *one* developer instance. Today, with modern DevOps and CI/CD multiple developers should be able to work on multiple feature branches simultaneously, in isolation. Developers should be able to run their test environments on their laptops, or in private cloud instances. Then these feature branches can be unit-tested, code-reviewed and pulled/merged into a staging/integration instance and finally pushed to production. This is modern best-practices, but SE's licensing model forbids this, requiring multiple expensive production licenses to legally enable it.
If you have improvements or suggestions please open an issue or pull request on the GitHub project page.
## Quick Test
For a quick smoke-test of social-nginx (without SocialEngine):
1. [Install Docker Desktop](https://www.docker.com/products/docker-desktop "Docker Desktop")
2. Open a terminal window and execute `docker run -p 80:80 tinkery/social-nginx`
3. Open a browser window to http://localhost. PHP-Info will be displayed.
## Running *Your* SocialEngine
Running SocialEngine in social-nginx the first time requires some setup. You must have shell access to your instance to set this up.
1. Install SocialEngine (or use your existing install- these instructions won't modify it). I haven't tested on a SocialEngine trial, but I don't see why it wouldn't work. Verify that it is running.
2. Temporarily switch SocialEngine to Development mode through the admin console.
3. Launch a shell (ie SSH) into your SocialEngine host.
4. Execute `cd /your/SocialEngine/root/ && tar czf ~/socialengine.tgz .`
5. Create a backup of your database. Note there is no space between -p and your password. Execute: `mysqldump -h YOUR_MYSQL_HOST -u root -pROOT_PASSWORD YOUR_DATABASE_NAME > ~/default.sql`
6. You'll want to download socialengine.tgz and default.sql from the home (~/) directory to your local computer. These instructions presume these are downloaded into ~/Downloads/socialengine.tgz and ~/Downloads/default.sql, but if they are in a different location you can just adjust the paths.
7. [install Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) if you don't have it.
8. Create a project directory: `mkdir ~/se-nginx-test && cd ~/se-nginx-text`
9. Clone social-ngix: `git clone https://github.com/TheTinkery/social-nginx.git`
10. (Optional) If you're going to store your site in GitHub, go create the repo now. Don't create a README in the project for now- just an empty project. This should be a private repository for holding your SocialEngine code.
11.  Create a local SocialEngine application directory: `mkdir socialengine`. Your *se-nginx-test* project directory should now contain two subdirectories: *social-ngix* and *socialengine*. Change into the *socialengine* directory: `cd socialengine`
12. (Optional) If you're going to store your SocialEngine site in a private GitHub repository, initialize Git to track file changes. Execute: `git init`
13. social-nginx includes an empty project structure you can start with. Execute: `cp -R ../social-nginx/`


### Versioning
| Docker Tag | Git Release | Nginx Version | PHP Version | Alpine Version |
|-----|-------|-----|--------|--------|
| latest/1.5.7 | Master Branch |1.14.0 | 7.2.10 | 3.7 |

For other tags please see: [versioning](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/versioning.md)

### Links
- [https://gitlab.com/ric_harvey/nginx-php-fpm](https://gitlab.com/ric_harvey/nginx-php-fpm)
- [https://registry.hub.docker.com/u/richarvey/nginx-php-fpm/](https://registry.hub.docker.com/u/richarvey/nginx-php-fpm/)

## Quick Start
To pull from docker hub:
```
docker pull richarvey/nginx-php-fpm:latest
```
### Running
To simply run the container:
```
sudo docker run -d richarvey/nginx-php-fpm
```
To dynamically pull code from git when starting:
```
docker run -d -e 'GIT_EMAIL=email_address' -e 'GIT_NAME=full_name' -e 'GIT_USERNAME=git_username' -e 'GIT_REPO=github.com/project' -e 'GIT_PERSONAL_TOKEN=<long_token_string_here>' richarvey/nginx-php-fpm:latest
```

You can then browse to ```http://<DOCKER_HOST>``` to view the default install files. To find your ```DOCKER_HOST``` use the ```docker inspect``` to get the IP address (normally 172.17.0.2)

For more detailed examples and explanations please refer to the documentation.
## Documentation

- [Building from source](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/building.md)
- [Versioning](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/versioning.md)
- [Config Flags](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/config_flags.md)
- [Git Auth](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/git_auth.md)
  - [Personal Access token](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/git_auth.md#personal-access-token)
  - [SSH Keys](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/git_auth.md#ssh-keys)
- [Git Commands](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/git_commands.md)
 - [Push](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/git_commands.md#push-code-to-git)
 - [Pull](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/git_commands.md#pull-code-from-git-refresh)
- [Repository layout / webroot](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/repo_layout.md)
 - [webroot](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/repo_layout.md#src--webroot)
- [User / Group Identifiers](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/UID_GID_Mapping.md)
- [Custom Nginx Config files](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/nginx_configs.md)
 - [REAL IP / X-Forwarded-For Headers](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/nginx_configs.md#real-ip--x-forwarded-for-headers)
- [Scripting and Templating](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/scripting_templating.md)
 - [Environment Variables](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/scripting_templating.md#using-environment-variables--templating)
- [Lets Encrypt Support](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/lets_encrypt.md)
 - [Setup](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/lets_encrypt.md#setup)
 - [Renewal](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/lets_encrypt.md#renewal)
- [PHP Modules](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/php_modules.md)
- [Xdebug](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/xdebug.md)
- [Logging and Errors](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/logs.md)

## Guides
- [Running in Kubernetes](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/guides/kubernetes.md)
- [Using Docker Compose](https://gitlab.com/ric_harvey/nginx-php-fpm/blob/master/docs/guides/docker_compose.md)
