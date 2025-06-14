# Cloudpanel and Netbird setup with Nginx reverse-proxy config

This repository contains setup and config files to create the Netbird self-hosted docker instances with Zitadel in combination with Cloudpanel running Nginx as a reverse-proxy.

[netbird-with-zitadel-without-caddy.sh](https://github.com/ggfx/cloudpanel-netbird-reverse-proxy/blob/main/netbird/netbird-with-zitadel-without-caddy.sh) is based on [getting-started-with-zitadel.sh](https://github.com/netbirdio/netbird/blob/main/infrastructure_files/getting-started-with-zitadel.sh), Copyright (c) 2022 NetBird GmbH & Authors.

## Requirements

* A public available virtual server e.g. Hetzner, DigitalOcean, etc. and running [Cloudpanel v2](https://www.cloudpanel.io/docs/v2/getting-started/).

* Also your VPS should meet the [self-host Netbird quickstart guide requirements](https://docs.netbird.io/selfhosted/selfhosted-quickstart#requirements). From my tests it is sufficient to open the TCP ports 80 and 443 in the firewall as well as UDP ports 3478 and 49152 to 65535.

* A domain you intend to use for your Netbird management endpoint e.g. _netbird.example.com_. Your domain A-record should be pointing to your VPS.

* [Docker](https://github.com/docker/docker-install) should already be installed. You can use this command from the server console to do so:
    ```sh
    cd /opt
    curl -fsSL https://get.docker.com -o get-docker.sh
    sh get-docker.sh
    ```

## Setup

If all the requirements are met, you can follow these steps:

1. Create a new **static website** in Cloudpanel with your _netbird.example.com_ domain. Go to Vhost and replace everything with the contents of [netbird/cloudpanel-netbird-reverse-proxy.conf](https://github.com/ggfx/cloudpanel-netbird-reverse-proxy/blob/main/netbird/cloudpanel-netbird-reverse-proxy.conf). Update `server_name YOUR_NETBIRD_DOMAIN;` with your `server_name netbird.example.com;` then click **Save**.
Switch to SSL/TLS and create a new Let's Encrypt Certificate for your domain.

1. If you can open your website in the browser and it shows "Hello World :-)" then you are able to proceed with the Netbird installation. Connect to your server via SSH and clone this repository:
    ```sh
    cd /opt
    git clone https://github.com/ggfx/cloudpanel-netbird-reverse-proxy.git
    ```
    Change into the cloudpanel-netbird-reverse-proxy/netbird directory and start the script:
    ```sh
    cd cloudpanel-netbird-reverse-proxy/netbird
    sudo chmod +x netbird-with-zitadel-without-caddy.sh
    ./netbird-with-zitadel-without-caddy.sh
    ```
    If you are asked for your NETBIRD_DOMAIN enter your _netbird.example.com_ domain.

1. Remove the Cloudpanel default index.html; Use your appropriate directory.
    ```sh
    rm /home/example-netbird/htdocs/netbird.example.com/index.html
    ```

1. Now you should reach netbird at https://netbird.example.com.

1. As a final security step you should consider changing the password for the default Zitadel Admin user. Open https://netbird.example.com/ui/console and login with _zitadel-admin@zitadel.netbird.example.com_ and the password = _Password1!_ (refer to [Self-hosting Zitadel](https://zitadel.com/docs/self-hosting/deploy/compose)). **This represents a potential security risk!**


# Known issues

There may be problems reaching the [turn server](https://github.com/netbirdio/netbird/issues/2567). 

Cloudpanel uses UFW firewall, so be sure to check your firewall first with
```sh
ufw status verbose
# or
# iptables -L
```
for the presence of:
```sh
3478/udp                   ALLOW IN    Anywhere
49152:65535/udp            ALLOW IN    Anywhere
```

## Possible solutions

* Adjust the UFW firewall yourself and allow the necessary UDP Ports 3478, 49152-65535:
    ```sh
    ufw allow 3478/udp
    ufw allow 49152:65535/udp
    ```

* You may turn off stun/turnserver completely, referring to https://github.com/netbirdio/netbird/issues/3546.

* Publish the ports in `docker-compose.yml` instead of using network_mode = host. Use only a few ports, otherwise Docker will not start or at least take a decade.

    Remove **network_mode: host** and enable **ports** instead in coturn service:
    ```env
    #network_mode: host
    ports:
      - "3478:3478/udp"
      - "65352-65535:65352-65535/udp"
    ```
    Adjust ports in `turnserver.conf`:
    ```env
    min-port=65352
    max-port=65535
    ```


# docker-compose-netbird-client.yml

This file is only for testing. Personally I stick to the native client because the docker container lacks DNS resolving. If you want use it anyway create your NB_SETUP_KEY in Netbird, copy `client.env.example` to `client.env` and set your variables, NB_SETUP_KEY can not be empty:
```env
NB_MANAGEMENT_URL=https://netbird.example.com
NB_SETUP_KEY=
```