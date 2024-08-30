# Unifi Network Application setup instructions
Unifi Network Application requires MongoDB 4.4 and earlier. These versions of MongoDB not included in apt repository for Ubuntu 22.04 and later. So we need some trick to install Unifi Network Application on Ubuntu 22.04.
## 1. Install prerequisites
1. Install packages
    ```bash
    apt-get update && apt-get install ca-certificates apt-transport-https net-tools jq socat iptables-persistent
    ```
1. Install `libssl 1.1` used by MongoDB 4.4
    ```bash
    wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_amd64.deb
    dpkg -i libssl1.1_1.1.1f-1ubuntu2_amd64.deb
    ```
1. Install `acme.sh`
    
    Start as root:
    ```bash
    curl https://get.acme.sh | sh -s email=you@mail.address
    ln -s /root/.acme.sh/acme.sh /usr/local/bin/
    ```
1. Add MongoDB 4.4 repo
    ```bash
    curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/mongodb-4.4.gpg
    echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
    ```

    > [!IMPORTANT]
    > Use `focal/mongodb-org/4.4` instead of nonexistent `jammy/mongodb-org/4.4`!

1. Add Unifi repo
    ```bash
    sudo wget -O /etc/apt/trusted.gpg.d/unifi-repo.gpg https://dl.ui.com/unifi/unifi-repo.gpg
    echo 'deb [ arch=amd64,arm64 ] https://www.ui.com/downloads/unifi/debian stable ubiquiti' | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list
    ```
1. Upgrade packages. Reboot if needed.
    ```bash
    apt-get update && apt upgrade
    ```
## 2. Install Unifi Network Application
1. Install Unifi Network Application
    ```bash
    apt-get install unifi
    ```
1. Setup port redirects

    Unifi Network Application can not use ports 80 and 443 itself because it started by unprivileged user. So we need redirect ports 80 and 443 to ports listened by UNA.
    ```bash
    iptables -t nat -I PREROUTING -p tcp --dport 443 -j REDIRECT --to-ports 8443
    iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080
    netfilter-persistent save
    ```
1. Issue Let's Encrypt SSL certificate
    
    I used ACME-DNS server for issue SSL certificate because my UNA is not accessible from Internet.

    First I requsted SSL certificate:
    ```bash
    export ACMEDNS_BASE_URL="https://<acme-dns-server>>"    
    acme.sh --issue --server letsencrypt --dns dns_acmedns -d una.domain.tld --log    
    ```
    Next I deployed issued certificate to UNA keystore:
    ```bash
    acme.sh --server letsencrypt --deploy --deploy-hook unifi -d una.domain.tld --log
    ```
