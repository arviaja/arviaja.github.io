# 2020-08-17-install-and-run-an-elasticsearch-instance-for-yourself

Elasticsearch is a really handy and nifty tool for everyone who wants to analyze large sets of data, find correlations and trends and use some AI functionality with a relatively small footprint.

## Let's get started
At the time of the writing of this post, Elastic is at version 7.8. Usually ELK (Elasticsearch, Logstash, Kibana) only introduce major changes in major version upgrades, so this should work just fine for all Versions 7.x.

Elastic runs on a number of platforms (https://www.elastic.co/support/matrix), for the sake of simplicity, I will pick Ubuntu 18.04 (20.04 is not supported).

Elastic has some advise on how to scale a cluster and once you go productive, you need to check all of this out. For testing and training purposes, if you're not running massive amounts of data, you should be fine with 2 virtual cpu's and 4 Gig of RAM, and of course some drive space (you want to use ssd if you are left with a choice).

We're also trying to save time, so we're going with the easiest installation process, which is using a repository (https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html#deb-repo)

I'm assuming you have a typical sodu-environment set up. If not and you're running everything as root (which you shouldn't !), just omit the ```sudo``` command. If I omit the command in these instructions, it is actually not required.

First we need to add the repository and the pgp-key (that first):

```
$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
$ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
```


## Elasticsearch

Then enable the service to start elastic at startup and right now:

```
$ sudo /bin/systemctl daemon-reload
$ sudo /bin/systemctl enable elasticsearch.service
```


## Kibana, NGINX + ssl-cert

```
$ sudo apt install nginx
$ sudo apt-get install apache2-utils
$ sudo htpasswd -c /etc/nginx/.kibana-user kibanauser
$ sudo nano /etc/nginx/sites-available/kibana
```

This should be the content of ```etc/nginx/sites-available/kibana```:

```
server {
    listen 80;
 
    server_name [your.server.tld];
 
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.kibana-user;
 
    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```
$ sudo ln -s /etc/nginx/sites-available/kibana /etc/nginx/sites-enabled
$ sudo nginx -t
$ sudo systemctl enable nginx
$ sudo systemctl restart nginx
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt install python-certbot-nginx
$ sudo certbot --nginx -d [your.kibana.server.hostname.tld]

```


## Logstash plus OpenJDK 8

You will need OpenJDK 8 even though Elasticsearch comes with a bundled Java Environment. However Logstash seems to not work properly with that, so just install it from the reps.

```
$ sudo apt-get install openjdk-8-jdk
$ sudo apt-get update && sudo apt-get install logstash
$ sudo systemctl enable logstash.service
$ sudo systemctl start logstash.service
```

