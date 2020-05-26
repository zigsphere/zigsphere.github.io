---
layout: post
title: Configuring Multiple Domains for a Single IP
tags: [Tech, How-to]
author: zigsphere
excerpt_separator: <!--more-->
---

I've been seeing a lot of questions regarding how to point multiple domains to one IP or how to configure Nginx or a similar service to listen on the same ports for multiple domains. I'm writing this post to say, yes, these are possible and many people and many companies do this each and every day.

Example questions I may hear:

> I have example.com and sample.domain.com and I want to point them to the same IP. Is this possible?

AND

> I have example.com and sample.domain.com. I am using Nginx and I want to use port 443 for both domains rather than a separate port. Is this possible?

The answer to both of these questions is YES. In fact, you can point a few hundred domains to a single IP if you really wanted to and there may be some interesting use-cases to point that many to a single IP, but today, I am going to use 4 domains as an example.

In the examples I'm using throughout this post, I will be using the following domains:

 1. site1.josephziegler.com
 2. site1.example.com
 3. site1.helloworld.com
 4. site1.zigsphere.com

Also, for example purposes, I will be using the Nginx service; however, other web services may support this capability. 

# SNI

SNI, Server Name Identifier, is the mechanism that allows multiple domains to point to a single IP. Services such as HAProxy or Nginx allow SNI and will allow serving a web application or service by the use of the server name identifier in their configuration files. Serving may consist of anything from running a static website, enabling reverse proxy, or any application using TLS. SNI is an extension to the Transport Layer Security; however, you can still use the server name identifier in non-TLS applications on non-ssl ports such as port 80 in HAProxy, for example, by using domain mapping. Since port 80 is not TLS, this technically wouldn't be SNI, but you can still map a domain to a backend.

To read more about SNI, you can navigate to [Here](https://en.wikipedia.org/wiki/Server_Name_Indication).

# DNS Configuration

For each domain, you will have an A Host DNS record to the same IP address. For example purposes, let's assume the public IP is 172.32.55.23

In your DNS provider, you would setup the DNS like so:
 1. site1.josephziegler.com -> A Record 172.32.55.23
 2. site1.example.com -> A Record 172.32.55.23
 3. site1.helloworld.com -> A Record 172.32.55.23
 4. site1.zigsphere.com -> A Record 172.32.55.23


# Nginx Configuration

Now let's get into how to configure SNI on Nginx. You may have been using SNI for quite some time, but perhaps you've never heard of the term. The parameter in Nginx is `server_name`. First of all, the sites themselves can either be hosted locally on the Nginx server or can reverse proxy to another web server or service on a different server in your environment. Nginx is serving as the SSL termination for your site or application. This is great for security purposes. First, let's explain the domains and what each one is doing.

### Overview

##### site1.josephziegler.com

site1.josephziegler.com is a website serving a static webpage on the Nginx server locally.

##### site1.example.com

site1.example.com is a website that has a nodejs backend. Nginx reverse proxies to another server, app.example.local, that is listening on port 5000 that is only accessible from the internal network and the Nginx server. 

##### site1.helloworld.com

site1.helloworld.com is a website that has a Java backend. It is a Java web service. Nginx reverse proxies to an internal load balancer, applb.helloworld.local (the load balancer is in front of 4 servers running on port 4000, serving the Java service).

##### site1.zigsphere.com

site1.zigsphere.com is a static website, the same as site1.josephziegler.com; however, this is serving a static website on another Nginx server. site1.josephziegler.com will be reverse proxying to port 80 on site1.zigsphere.local, which is only accessible internally by internal nodes and the site1.zigsphere.com Nginx server.


### Config Setup

Assuming Nginx is installed, you have a directory `/etc/nginx/sites-enabled`. For each domain we are setting up, there will be a separate config file. When Nginxis installed, there may be a `default` file, `/etc/nginx/sites-enabled/default`. Either delete this or rename to one of the filenames shown below. 

For each of our domains, let's name the filenames as such:

 - site1.josephziegler.com -> `/etc/nginx/sites-enabled/josephziegler.conf`
 - site1.example.com -> `/etc/nginx/sites-enabled/example.conf`
 - site1.helloworld.com -> `/etc/nginx/sites-enabled/helloworld.conf`
 - site1.zigsphere.com -> `/etc/nginx/sites-enabled/zigsphere.conf`

 Now, let's configure each of the config files shown above:

###### /etc/nginx/sites-enabled/josephziegler.conf


```
server {
	listen *:80;

	root /var/www/html;
	index index.html index.htm;
	server_name site1.josephziegler.com;    
	return 301 https://$host$request_uri;
	access_log /var/log/nginx/site1.josephziegler.com.access.log combined;
	error_log  /var/log/nginx/site1.josephziegler.com.error.log;
	
}

server {
  listen       *:443 ssl;

  server_name  site1.josephziegler.com;

  ssl on;
  ssl_certificate /etc/letsencrypt/live/site1.josephziegler.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/site1.josephziegler.com/privkey.pem; # managed by Certbot
  ssl_dhparam               /etc/ssl/certs/dhparams.pem;
  ssl_session_cache         shared:SSL:50m;
  ssl_session_timeout       5m;
  ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers               ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
  ssl_prefer_server_ciphers on;
  ssl_stapling              on;
  ssl_stapling_verify       on;
  ssl_trusted_certificate   /etc/letsencrypt/live/site1.josephziegler.com/cert.pem;
  index  index.html;

  access_log            /var/log/nginx/ssl-site1.josephziegler.com.access.log combined;
  error_log             /var/log/nginx/ssl-site1.josephziegler.com.error.log;
  root /var/www/html;

  location / {
    index  index.html;
    root /var/www/html;
  }

}
```

For this first one, let's go over the config a little bit. 

 - For each domain, we are listening on both 80 and 443. For port 80, we are redirecting to 443 so that each domain serves SSL traffic. This is done by doing a `return 301 https://$host$request_uri;` 
 - For each domain, we are logging to a separate log file. This is essential so we dont have every domain logging to the SAME config file. This would be incredible hard to troubleshoot if there was an issue.
 - For each domain, we are specifying it's own SSL certificate directory such as `/etc/letsencrypt/live/site1.josephziegler.com/fullchain.pem;`l however, if you have multiple domains on one Nginx host, you will likely only have 1 main directory for ALL domains if you are using Let's Encrypt, so you would specify that here.
 - Finally, the location directive, `location /`, which tells Nginx what we are serving. In this case, we are just telling it to look for the index.html file located in /var/www/html. You can set this path to whatever you want. 

###### /etc/nginx/sites-enabled/example.conf

```
server {
	listen *:80;

	root /var/www/html;
	index index.html index.htm;
	server_name site1.example.com;    
	return 301 https://$host$request_uri;
	access_log /var/log/nginx/site1.example.com.access.log combined;
	error_log /var/log/nginx/site1.example.com.error.log;
	
}

server {
  listen       *:443 ssl;

  server_name  site1.example.com;

  ssl on;
  ssl_certificate /etc/letsencrypt/live/site1.example.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/site1.example.com/privkey.pem; # managed by Certbot
  ssl_dhparam               /etc/ssl/certs/dhparams.pem;
  ssl_session_cache         shared:SSL:50m;
  ssl_session_timeout       5m;
  ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers               ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
  ssl_prefer_server_ciphers on;
  ssl_stapling              on;
  ssl_stapling_verify       on;
  ssl_trusted_certificate   /etc/letsencrypt/live/site1.example.com/cert.pem;
  index  index.html index.htm index.php;

  access_log            /var/log/nginx/ssl-site1.example.com.access.log combined;
  error_log             /var/log/nginx/ssl-site1.example.com.error.log;
  root /var/www/html;

  location / {
    proxy_pass http://app.example.local:5000;
    proxy_set_header User-Agent $http_user_agent;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Accept-Encoding "";
    proxy_set_header Accept-Language $http_accept_language;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

}
```

As shown above, this is the same config as josephziegler.conf; however, the domain is different for `server_name` and set to the site1.example.com domain, the SSL directory is different and set to it's own files, log directories are in their own directory, and the location directive is set to reverse proxy to the internal app.example.local server running the Nodejs application on port 5000.

###### /etc/nginx/sites-enabled/helloworld.conf

```
server {
	listen *:80;

	root /var/www/html;
	index index.html index.htm;
	server_name site1.helloworld.com;    
	return 301 https://$host$request_uri;
	access_log /var/log/nginx/site1.helloworld.com.access.log combined;
	error_log /var/log/nginx/site1.helloworld.com.error.log;
	
}

server {
  listen       *:443 ssl;

  server_name  site1.helloworld.com;

  ssl on;
  ssl_certificate /etc/letsencrypt/live/site1.helloworld.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/site1.helloworld.com/privkey.pem; # managed by Certbot
  ssl_dhparam               /etc/ssl/certs/dhparams.pem;
  ssl_session_cache         shared:SSL:50m;
  ssl_session_timeout       5m;
  ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers               ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
  ssl_prefer_server_ciphers on;
  ssl_stapling              on;
  ssl_stapling_verify       on;
  ssl_trusted_certificate   /etc/letsencrypt/live/site1.helloworld.com/cert.pem;
  index  index.html index.htm index.php;

  access_log            /var/log/nginx/ssl-site1.helloworld.com.access.log combined;
  error_log             /var/log/nginx/ssl-site1.helloworld.com.error.log;
  root /var/www/html;

  location / {
    proxy_pass http://applb.helloworld.local:4000;
    proxy_set_header User-Agent $http_user_agent;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Accept-Encoding "";
    proxy_set_header Accept-Language $http_accept_language;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

}
```

###### /etc/nginx/sites-enabled/zigsphere.conf

```
server {
	listen *:80;

	root /var/www/html;
	index index.html index.htm;
	server_name site1.zigsphere.com;    
	return 301 https://$host$request_uri;
	access_log /var/log/nginx/site1.zigsphere.com.access.log combined;
	error_log /var/log/nginx/site1.zigsphere.com.error.log;
	
}

server {
  listen       *:443 ssl;

  server_name  site1.zigsphere.com;

  ssl on;
  ssl_certificate /etc/letsencrypt/live/site1.zigsphere.com/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/site1.zigsphere.com/privkey.pem; # managed by Certbot
  ssl_dhparam               /etc/ssl/certs/dhparams.pem;
  ssl_session_cache         shared:SSL:50m;
  ssl_session_timeout       5m;
  ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers               ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
  ssl_prefer_server_ciphers on;
  ssl_stapling              on;
  ssl_stapling_verify       on;
  ssl_trusted_certificate   /etc/letsencrypt/live/site1.zigsphere.com/cert.pem;
  index  index.html index.htm index.php;

  access_log            /var/log/nginx/ssl-site1.zigsphere.com.access.log combined;
  error_log             /var/log/nginx/ssl-site1.zigsphere.com.error.log;
  root /var/www/html;

  location / {
    proxy_pass http://site1.zigsphere.local:80;
    proxy_set_header User-Agent $http_user_agent;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Accept-Encoding "";
    proxy_set_header Accept-Language $http_accept_language;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

}
```

# Infrastructure Workflow

Below you will see how this would be setup in our examples. Several domains configured in Nginx and each of them doing independent things. 

<center><img src="https://www.josephziegler.com/media/many_domains_one_ip.png" alt="oneip"></center>

# Summary

As shown in my examples, you should now have an understanding about how to configure multiple domains to point at one domain using DNS and Nginx SNI. The ports open for each domain can be the same (80 and 443 in my example) and each domain has it's own configuration. 


