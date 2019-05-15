# Running system locally

## User Account on GitLab

Since you are reading this documentation, it's fair to assume that you already have a [gitLab.com](https://gitlab.com) user account. And that you've been given access to the `MUSIT-Norway` organisation.

#### 2-factor authentication

It is **highly** recommended that you enable 2-factor auth for your gitlab login.

#### GitLab API token

It is **highly** recommended that you create a [personal access token](https://gitlab.com/profile/personal_access_tokens). This is then used instead of your regular password when communicating with the gitlab.com API's, gitlab.com docker registries, etc.

After creating an access token, _make sure you copy the token and paste it somewhere secure_. Once you leave the page, it is not possible to read the token again.

Using a personal access token will ensure that your password isn't sent across the wire when calling the gitlab API's.

**If you have enabled 2-factor auth, using an access token is required!**

#### Add an SSH key

To make it easier to work with git on your local machine, it is highly recommended that you add an [ssh key](https://gitlab.com/profile/keys) to your user account. GitLab provides detailed documentation on [how to generate ssh keys](https://gitlab.com/help/ssh/README).

## Prerequisites

All the following chapters assume that all required items listed below are installed.

### Software

All the software listed below should be installable through the package manager of your OS.

#### Required

* Docker (and docker-compose)
* Nginx
* Node (v 7.5.0)
* npm
* OpenJDK 1.8.x (the backend system is _not_ using the Oracle JDK)
* SBT (Scala Build Tool)
* git


#### Recommended

The following lists some recommended tools that could boost your productivity. It is highly recommended to spend a little time and effort to become familiar with these.

* **zsh** (instead of bash) - zsh is a _much_ more powerful and convenient shell environment. A very good way to get started is to install [oh-my-zsh](http://ohmyz.sh).
* **httpie** (instead of curl or wget). Dead simple, but powerful, http client allowing to perform HTTP requests from the CLI.

## Cloning the relevant repositories

For this document we're going to assume there is a folder called `projects` in the home folder on your computer. Where you choose to call it is up to you.

```bash
# Go to the projects directory
cd ~/projects

# Clone the backend repository
git clone git@gitlab.com:MUSIT-Norway/musit.git

# Clone the frontend repository
git clone git@gitlab.com:MUSIT-Norway/musit-frontend.git
```

## Configure Nginx

#### Install `nginx` like described here:

```bash
sudo apt-get update
sudo apt-get install nginx
sudo update-rc.d nginx enable
```

####  Edit the `nginx.conf` file

As described in the url above, on Ubuntu the config resides in `/etc/nginx/nginx.conf`. Open this in a suitable editor, like `sudo gedit /etc/nginx/nginx.conf`, mark and delete everything, and copy in the configuration settings below. Save the file.

```bash
worker_processes  4; # Mind this number...no more than 2 workers per core.

events {
  worker_connections  19000;
}

worker_rlimit_nofile 20000;

http {
  include       mime.types;
  default_type  application/octet-stream;

  sendfile        on;
  keepalive_timeout  65;

  client_max_body_size 2048M;

  gzip on;
  gzip_comp_level 1;
  gzip_min_length 1000;
  gzip_types  text/plain application/javascript application/x-javascript text/javascript text/xml text/css ;

  # Add a vary header for downstream proxies to avoid sending cached gzipped files to IE6
  gzip_vary on;

  # For the geeks: "A man is not dead while his name is still spoken." -Terry Pratchett
  add_header X-Clacks-Overhead "GNU Terry Pratchett";

  # Shared proxy settings
  proxy_no_cache 1;

  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header Host $host;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-NginX-Proxy true;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";

  proxy_http_version 1.1;
  proxy_read_timeout 1200s;

  # Turn off proxy buffering...
  proxy_request_buffering off;
  proxy_buffering off;

  server {
    listen       8888;
    server_name  localhost;

    location /service_auth {
      proxy_pass http://localhost/service_auth;
    }

    location /api {
      proxy_pass http://localhost/api;
    }

    location / {
      proxy_pass http://localhost:3000;
    }

    # redirect server error pages to the static page /50x.html
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
      root   html;
    }
  }
}
```

#### Restart nginx

Execute the following command to restart nginx on Ubuntu:

```bash
sudo systemctl restart nginx
```

## Configure the `hosts` file

The hosts file must be edited for your local deployment to work properly. This is necessary to make sure the application uses a pre-configured application config in dataporten.

1. Locate the `hosts` file for your operating system. For *nix based OS'es you can find it in `/etc/hosts`.

2. Open the the `hosts` file with your favorite editor.

    ```bash
    sudo vi /etc/hosts
    ```
3. Add the following entry to the bottom of the file

    ```bash
    127.0.0.1	localhost  musit-test
    ```


## Starting the backend services


### Starting backend services in docker

```bash
# First we navigate into the backend project
cd ~/project/musit

# Then go to the docker/dev folder
cd docker/dev

# Execute the deploy.sh script
./deploy.sh
```

The `deploy.sh` script will perform a series of commands to build and publish the docker images for each backend service locally. Once that is done, they are started using `docker-compose`.



To verify that all the necessary docker containers are running, execute the following command:

```bash
docker ps
```


### Starting the frontend application

First we need to prepare and start the actual frontend application:

```bash
# First we navigate into the frontend project
cd ~/project/musit-frontend

# Install dependencies
npm install

# Start the frontend application
BROWSER=false npm start
```

Once the application is ready, you can navigate to [http://musit-test:8888](http://musit-test:8888) in your web browser. Where you'll be taken to the login screen of the MUSIT system.

