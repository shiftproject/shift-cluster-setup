# Shift Cluster Setup

This script can be used to setup and install the IPFS daemon and Phoenix cluster daemon (IPFS pin manager) for Shift.

## Prerequisites

*Note: This has only been tested on Ubuntu. Make sure you are using a clean install. If you had previously installed shift-cluster for testing purposes you will have to run `shift-cluster remove` before `shift-cluster install`.*

Before running the script you should make sure your system is up to date and install dependencies as root:

```
apt-get update && apt-get -y upgrade && apt-get -y dist-upgrade
apt-get -y install git nano ufw build-essential jq
```

You can optionally create a shift user to run as

```
useradd -s /bin/bash -m shift
```

Give the shift user sudo access by running `sudo visudo` and adding `shift ALL=(ALL) NOPASSWD:ALL` to the sudoers file.

Then switch to the shift user

```
sudo su - shift
```

And make sure the following ports are open on the firewall:

```
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 4001/tcp
sudo ufw allow 3473/tcp
sudo ufw enable
```

## Installation

To install or update the `shift-cluster` command on linux run the following:

```
curl https://install.shiftproject.com | env=testnet bash
```

This will place it in the bin directory of your home folder and make it executable. Make sure that `$HOME/bin` is included in your $PATH.

After this manager script is installed to actually install the binaries and join the cluster you will have to run

```
shift-cluster install
```

then

```
shift-cluster start
```

Note: If you already have a webserver listening on ports 80/443 on your server, please follow the instructions below (Nginx only, but can be adapted to Apache2)

## (Optional) Using Nginx as a front reverse proxy

Shift-cluster uses haproxy has a backend and frontend provider. The frontend listens on *:80 and *:443. In order to keep using these ports for other purposes/projets, you can put haproxy behind a reverse proxy using Nginx

(Note: We assume you already have a Nginx server up and running)

First step is to change the `/etc/haproxy/haproxy.cfg` configuration, by editing these two lines after `frontend https-in` :

```
frontend https-in
    #bind *:443 ssl crt /etc/ssl/private/shift.pem
    bind 127.0.0.1:8081
```

Here we remove haproxy SSL management for HTTPS (it will be handled by Nginx later), and we change the port 80 to 8081 (or any other port you have available, this port will be listening only on localhost, so there is no need to open any firewall rule)

Now you can restart haproxy : `sudo service restart haproxy`

Then, we need to create a nginx vhost that will forward the traffic from ports 80/443 to 127.0.0.1:8081 (or the port you've chosen)

Here is an example configuration (create a file `/etc/nginx/conf.d/phoenix.conf`) , you will have to change the haproxy port you defined earlier (8081 if you followed this documentation) and the lines containing `server_name IP_OR_DOMAIN_NAME` by using your node domain/subdomain name (If you don't have one, just remove the server_name line entirely, your vhost will be reachable through your IP address then, but HTTPS might not work properly)

### Method 1 : HTTP Vhost only (not recommended if you use a dedicated domain name for your phoenix cluster)

```
server {
    listen *:80;
    server_name DOMAIN_NAME;

    access_log /var/log/nginx/phoenix_http_access.log;
    error_log /var/log/nginx/phoenix_http_error.log;

    root /var/www/html;
    index index.html index.htm index.php;

    location  / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```


### Method 2 : HTTPS Vhost with HTTP forwarding to HTTPS

Please note that in order to have a valid HTTPS behaviour, you will have to generate certificates for a domain name (using Letsencrypt) and edit the directives `ssl_certificate PATH_TO_FULLCHAIN.PEM;` and `ssl_certificate_key PATH_TO_PRIVKEY.PEM;`.

You can also adapt if you don't have a domain name with a self-signed certificate, but modern web browser will give you a warning :(

```
## Redirects all HTTP traffic (80) to the HTTPS host (443)
server {
  listen *:80;
  server_name DOMAIN_NAME;
  server_tokens off; ## Don't show the nginx version number, a security best practice
  return 301 https://$http_host$request_uri;
  access_log  /var/log/nginx/phoenix_http_access.log;
  error_log   /var/log/nginx/phoenix_http_error.log;
}

server {
    listen *:443 ssl;
    server_name DOMAIN_NAME;
    ssl_certificate PATH_TO_FULLCHAIN.PEM;
    ssl_certificate_key PATH_TO_PRIVKEY.PEM;

    access_log /var/log/nginx/phoenix_https_access.log;
    error_log /var/log/nginx/phoenix_https_error.log;

    root /var/www/html;
    index index.html index.htm index.php;

    location  / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header HTTPS   $https;
    }
}
```

## Verifying installation

If the installation is successful and you have joined your IP address should show up in the list at https://storage-testnet.shiftproject.com/peers with `"Online": true`.

Also `shift-cluster check` should report that IPFS is running, and `ipfs swarm peers` should list some other peers in the cluster. You can check if it is able to deliver content by requesting content directly from your IP:

```
http://{IP_ADDRESS}/ipns/shiftproject.com
```

### Commands

These are the available commands from `shift-cluster`.

#### Install

```
shift-cluster install
```

This installs the latest IPFS and phoenix-cluster binaries to run a Shift storage node.

#### Update

```
shift-cluster update
```

This will ensure that the latest IPFS and phoenix-cluster binaries are up to date, and if they are not, it will download the latest ones.

Note that it will not immediately start running the new code. For that you will need to run `shift-cluster restart`.

#### Remove

```
shift-cluster remove
```

Removes all of the files installed as part of the installation process. Note that this will also remove any content pinned in your IPFS storage.

#### Start

```
shift-cluster start
```

Starts up the IPFS daemon and cluster instance, connects to the Shift cluster, and starts up a local proxy to forward HTTP requests.

#### Stop

```
shift-cluster stop
```

Stops the proxy and IPFS instance and disconnects from the cluster.

#### Restart

```
shift-cluster restart
```

Stops then starts the running instance.

#### Check

```
shift-cluster check
```

Checks if IPFS is running.

## Help

If you are running into issues and need help, jump into the Shift discord channel: https://discord.gg/fgzxABX
