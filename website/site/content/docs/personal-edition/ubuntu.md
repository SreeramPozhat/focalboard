---
title: "Personal Server (Ubuntu)"
date: "2020-12-15T12:02:23-04:00"
subsection: Personal Edition
weight: 2
---

Karmaboard Personal Server is a standalone server for development and personal use. For team use, check out [Mattermost Boards](../../mattermost/), which supports private boards, team communication, and more.

Follow these steps it up on an Ubuntu server. To upgrade an existing installation, see [the upgrade guide](../ubuntu-upgrade).

## Set up Ubuntu Server 18.04

Popular hosted options include:
* [Digital Ocean](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04)
* [Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [Linode](https://www.linode.com/docs/products/compute/)

## Install Karmaboard

Download the Ubuntu archive package from the appropriate [release in GitHub](https://github.com/mattermost/karmaboard/releases). The example below uses the link for **v0.15.0**, but you're encouraged to use the latest version in the release list:

```
wget https://github.com/mattermost/karmaboard/releases/download/v0.15.0/karmaboard-server-linux-amd64.tar.gz
tar -xvzf karmaboard-server-linux-amd64.tar.gz
sudo mv karmaboard /opt
```

## Install NGINX

By default, the Karmaboard server runs on port 8000 (specified in config.json). We recommend running NGINX as a web proxy to forward http and websocket requests from port 80 to it. To install NGINX, run:

```
sudo apt update
sudo apt install nginx
```

You may need to adjust your firewall settings depending on the host, e.g.
* [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)
* [EC2](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/ec2-instances-for-nginx/)
* [Linode](https://www.linode.com/docs/guides/how-to-install-nginx-ubuntu-18-04/)

### Configure NGINX

Create a new site config:

```
sudo nano /etc/nginx/sites-available/karmaboard
```

Copy and paste this configuration:

```
upstream karmaboard {
   server localhost:8000;
   keepalive 32;
}

server {
   listen 80 default_server;

   server_name karmaboard.example.com;

   location ~ /ws/* {
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       client_max_body_size 50M;
       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Frame-Options SAMEORIGIN;
       proxy_buffers 256 16k;
       proxy_buffer_size 16k;
       client_body_timeout 60;
       send_timeout 300;
       lingering_timeout 5;
       proxy_connect_timeout 1d;
       proxy_send_timeout 1d;
       proxy_read_timeout 1d;
       proxy_pass http://karmaboard;
   }

   location / {
       client_max_body_size 50M;
       proxy_set_header Connection "";
       proxy_set_header Host $http_host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Frame-Options SAMEORIGIN;
       proxy_buffers 256 16k;
       proxy_buffer_size 16k;
       proxy_read_timeout 600s;
       proxy_cache_revalidate on;
       proxy_cache_min_uses 2;
       proxy_cache_use_stale timeout;
       proxy_cache_lock on;
       proxy_http_version 1.1;
       proxy_pass http://karmaboard;
   }
}
```

If there is a default site, you may need to delete it

```
sudo rm /etc/nginx/sites-enabled/default
```

Enable the Karmaboard site, test the config, and reload NGINX:

```
sudo ln -s /etc/nginx/sites-available/karmaboard /etc/nginx/sites-enabled/karmaboard
sudo nginx -t
sudo /etc/init.d/nginx reload
```

## Set up TLS on NGINX

For a production server, it's important to set up TLS to encrypt web traffic. Without this, your login passwords and data are unprotected. Refer to the [NGINX TLS guide](https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/) and [Let's Encrypt Certbot guide](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx) on setting this up.

## Install Postgresql (Recommended)

Karmaboard stores data in a SQLite database by default, but we recommend running against Postgres in production (we've tested against Postgres 10.15). To install, run:

```
sudo apt install postgresql postgresql-contrib
```

Then run as the postgres user to create a new database:

```
sudo --login --user postgres
psql
```

On the psql prompt, run the following commands (**change the user/password** to your own values):

```
CREATE DATABASE boards;
CREATE USER <b>boardsuser</b> WITH PASSWORD '<b>boardsuser-password</b>';
\q
```

Exit the postgres user session:

```
exit
```

Edit the Karmaboard config.json:

```
nano /opt/karmaboard/config.json
```

Change the dbconfig setting to use the postgres database you created:

```
"dbtype": "postgres",
"dbconfig": "postgres://boardsuser:boardsuser-password@localhost/boards?sslmode=disable&connect_timeout=10",
```

## Install MySQL

As an alternative to Postgres, you also can store your data in a MySQL database. To install, run:

```
sudo apt-get install mysql-server
```

Log in as `root` in your database:

```
sudo mysql
```

At the MySQL prompt, run the following commands (change `user/password` to your own values):

```
CREATE DATABASE boards;
GRANT ALL on boards.* to 'boardsuser'@'localhost' identified by 'boardsuser-password';
```

Exit the mysql-prompt:

```
exit
```

Edit the Karmaboard `config.json`:

```
nano /opt/karmaboard/config.json
```

Change the dbconfig setting to use the MySQL database you created:

When MySQL is being used, using collation is recommended over using charset.

Using a variant of `utf8mb4` collation is required. For example, `utf8mb4_general_ci`
is used by default when no collation is specified.

If you're using Karmaboard as a Mattermost Plugin prior to version 0.9 with MySQL,
please ensure the collations of karmaboard tables (tables with the prefix `karmaboard_`)
is the same as the collation of mattermost tables.

```
"dbtype": "mysql",
"dbconfig": "boardsuser:boardsuser-password@tcp(127.0.0.1:3306)/boards",
```

## Configure Karmaboard to run as a service

This will keep the server running across reboots. First, create a new service config file:

```
sudo nano /lib/systemd/system/karmaboard.service
```

Paste in the following:

```
[Unit]
Description=Karmaboard server

[Service]
Type=simple
Restart=always
RestartSec=5s
ExecStart=/opt/karmaboard/bin/karmaboard-server
WorkingDirectory=/opt/karmaboard

[Install]
WantedBy=multi-user.target
```

Make systemd reload the new unit, and start it on machine reboot:

```
sudo systemctl daemon-reload
sudo systemctl start karmaboard.service
sudo systemctl enable karmaboard.service
```

## Test the server

At this point, the Karmaboard server should be running.

Test that it's running locally with:

```
curl localhost:8000
curl localhost
```

The first command checks that the server is running on port 8000 (default), and the second checks that NGINX is proxying requests successfully. Both commands should return the same snippet of HTML.

To access the server remotely, open a browser to its IP address or domain.

## Set up the server

Refer to the [server setup guide](/guide/server-setup/) to complete server setup.
