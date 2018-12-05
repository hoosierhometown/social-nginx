# Overview
This is a Docker image / container for running SocialEngine on NGINX with PHP-FPM.

This repository does **not** contain any SocialEngine components. Rather, it is a container configured to run SocialEngine and support DevOps workflows. It is designed to work with a SocialEngine instance (code and database) stored in Github. Note that because SocialEngine is a commercial product, it should not be stored in a public repository. Github supports private repositories for individuals and organizations, though you need a paid account to take advantage of them. You can test this without a Github repository.

This image supports modern DevOps workflows around SE, using a Docker container for the application environment, and loading your SocialEngine site code and database from Github. Using Docker Compose, you can run a fully self-contained development environment on your laptop, push changes to source control and have them instantly up and running on a container-based server. I've currently got this running on the Google Cloud Platform Compute Engine, using a VPS that is initialized with the social-nginx image.

This initial version is a work in progress. I currently don't have SSL set up (though that shouldn't be difficult). I'm currently in early development on a SocialEngine site so I'll be updating this as I go. I would welcome others to fork this repository, and contribute your improvements through pull requests. If you experience problems, you may also open an issue on the Github project page.
## Licensing
SocialEngine is licensed in an old-school "paid script" model where everybody hacks on one instance of the script running on some VPS. To support this old model SE allows only *one* developer instance. Today, with modern DevOps and CI/CD multiple developers should be able to work on multiple feature branches simultaneously, in isolation. Developers should be able to easily and quickly run their dev environments on their laptops, or in private cloud instances. Then these feature branches can be unit-tested, code-reviewed and pulled/merged into a staging/integration instance and finally pushed to production. This is modern best-practices, but SE's licensing model forbids this, requiring multiple expensive production licenses to legally enable it. I would highly encourage SE to adopt a licensing approach that doesn't discourage developers from using modern, efficient DevOps.
## Improving social-nginx
If you have improvements or suggestions please open an issue or pull request on the GitHub project page.
# Quick Test
For a quick smoke-test of social-nginx (without SocialEngine):
1. [Install Docker Desktop](https://www.docker.com/products/docker-desktop "Docker Desktop")
2. Open a terminal window and execute `docker run -p 80:80 tinkery/social-nginx`
3. Open a browser window to http://localhost. PHP-Info will be displayed.
# Running SocialEngine
Running SocialEngine in social-nginx the first time requires some setup. You must have shell access to your instance to set this up.
1. Install SocialEngine (or use your existing install- these instructions won't modify it). I haven't tested on a SocialEngine trial, but I don't see why it wouldn't work. Verify that it is running.
2. Temporarily switch SocialEngine to Development mode through the admin console.
3. Launch a shell (ie SSH) into your SocialEngine host.
4. Execute `cd /your/SocialEngine/root/ && tar czf ~/socialengine.tgz .`
5. Create a backup of your database. Note there is no space between -p and your password.
Execute: `mysqldump -h YOUR_MYSQL_HOST -u root -pROOT_PASSWORD YOUR_DATABASE_NAME > ~/default.sql`
6. You can set your site back to production mode.
7. Download socialengine.tgz and default.sql from the home (~/) directory to your local computer. These instructions presume these are downloaded into ~/Downloads/socialengine.tgz and ~/Downloads/default.sql, but if they're in a different location you can just adjust the paths.
8. [Install Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) if you don't have it.
9. Create a project directory: `mkdir ~/se-nginx-test && cd ~/se-nginx-test`
10. Clone social-ngix: `git clone https://github.com/TheTinkery/social-nginx.git`
11. (Optional) If you're going to store your site in GitHub, go create the repo now. Don't create a README.md or .gitignore in the project for now- just an empty project. This should be a private repository for holding your SocialEngine code.
12.  Create a local SocialEngine application directory: `mkdir socialengine`. Your *se-nginx-test* project directory should now contain two subdirectories: *social-ngix* and *socialengine*. Change into the *socialengine* directory: `cd socialengine`
13. (Optional) If you're going to store your SocialEngine site in a private GitHub repository, initialize Git to track file changes. Execute: `git init`
14. Social-nginx includes an empty project structure you can start with. Execute: `cp -R ../social-nginx/site-template/. .`. This structure includes a .gitignore that ignores Mac's .DS_Store and also excludes the SocialEngine temporary directory.
15. Extract your site source code (modify the download path if necessary): `cd src && tar xzf ~/Downloads/socialengine.tgz && cd ..`
16. Copy the database backup (modify the download path if necessary): `cp ~/Downloads/default.sql conf/db/`. Your project should now be ready to run.
17. Pre-fetch a couple Docker images: `docker pull mariadb && docker pull tinkery/social-nginx`
18. Change into the dev directory `cd dev`
19. Note: We're going to run a local MySQL database (technically, it's MariaDB). The configuration includes a root password and a user and password for a SocialEngine database. This is a development only database instance. You can leave these as-is, or change to your preference by editing docker-compose.yml. It isn't necessary that this matches your database.
20. Make the scripts excecutable: `chmod +x start stop enter enterdb`
21. Execute `./start` to launch the mariadb and social-nginx containers using the SocialEngine code you have set up. Startup may take a couple minutes. If you execute `docker ps` you should see the two containers running. If you experience trouble, you can execute `./stop` to shutdown these instances, then execute `docker-compose up` to see startup messages from the containers.
22. Executing `./enter` will give you a shell inside the social-nginx container.  Executing `./enterdb` will give you a shell inside the mariadb container.
23. Launch your browser to access your site at http://localhost.
24. You can edit your site files directly in the src directory. If you make changes through the admin panel (such as installing plugins) these changes will be in your local src directory as well. Note that you may need to tweak directory permissions in your docker container when installing plugins or themes. I will work on fixing this up later.
25. If you make changes that impact your database (most do) you need to export your database so it will be saved in source control. To do this, execute: `docker exec social-nginx backupdb`. This will backup the database to conf/db/default.sql.
26. Shut down: `./stop`
27. (Optional) If storing your site in a private GitHub project, you can push it now:
This prevents file mode changes from being tracked by Git (only need to execute this once):

```
git config --global push.default simple
git config --global core.fileMode false
```

Then add, commit and push your changes

```
cd ..
git add -A
git commit -a -m "Initial commit"
git push origin master`
```

Now it's extremely simple to run your SocialEngine site anywhere. Just clone your repository, cd into the dev directory, then run ./start. Developers can work on feature branches in total isolation, allowing you to merge the changes when they've been completed and tested.

Note: dev/fromgit/ contains a docker-compose.yml file that includes repository info. If you edit this file to set your Git account details, then run `docker compose up -d` in this directory, social-nginx will fetch a copy of your SocialEngine site from Github when it launches. You must set up a personal access token for this to work (it is easy to set this up in Github). When you launch this way, you are not working on a local copy of your code. This mode is especially useful for running your site in production or staging, or on a Docker or Kubernetes container in the cloud. When running in this mode, `docker exec social-nginx pull` will fetch updates from the repository, and `docker exec social-nginx push` will push changes from the docker image to the repository.

Note 2: The .gitignore causes github to exclude SocialEngine's temporary directory. It's my presumption that this directory is only for transient objects. However, I wasn't sure if SE was smart enough to always re-create the directory structure it needs in temporary. So the startup script recreates the structure here that existed on my installation. If anybody knows what I should do differently here - either files I need to preserve in temporary, or if SE will automatically create any directories it needs here, let me know and I can clean this up.
## Next steps
* Implement SSL
* Optimize and clean up NGINX & PHP config. Perhaps break PHP-FPM into a separate container? Make sure there's no rewrite issues.
* Incorporate other PHP modules as needed
* Optimize & harden directory permissions for dev & production.
* Test on GCP Kubernetes Engine
* Implement a test harness for automated testing
* Implement a database migration tool, supporting continuous delivery all the way into production, including schema and data changes.
* Automate CI/CD using Jenkins or a cloud DevOps pipeline
* Test on AWS and Azure
* Update social-nginx docs.

This will result in full Continuous Integration and Delivery of even large SocialEngine installations, while enabling developers to easily work on isolated feature branches.
