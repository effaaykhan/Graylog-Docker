# Graylog-Docker
Graylog is an open-source log management platform that allows users to collect, index, and analyze log data from various sources. It is designed to help organizations gain insights from their log data, troubleshoot issues, and ensure the security and compliance of their systems

## Install Docker.io on your system
```bash
sudo apt update
```
```bash
sudo apt install docker.io -y
```
```bash
sudo usermod -aG docker $USER
newgrp docker
docker --version
```
- Start and enable the docker
  ```bash
  systemctl enable docker
  systemctl start docker
  ```   
## Install Docker Compose 
- Install the dependencies
  ```bash
  sudo apt install -y curl git ca-certificates
  ```
- Now proceed with the installation of docker-compose
  ```bash
  sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  ```
- Set the right permissions
  ```bash
  sudo chmod +x /usr/local/bin/docker-compose
  ```
- Confirm the installation
  ```bash
  docker-compose --version
  ```

## Provision the graylog Container
- The graylog container will consist of the graylog server, elasticsearch and Mongodb. To be able to achieve this, we will capture the information and settings into a YML file.
- Make a separate directory to store the docker-compose.yml file
  ```bash
  mkdir graylog
  cd graylog
  ```
- Create a docker-compose.yml file
  ```bash
  nano docker-compose.yml
  ```
- In the docker-compose.yml file add the desired content. To get the docker-compose.yml content click [here](https://github.com/effaaykhan/Graylog-Docker/blob/main/docker-compose.yml)
- Create .env file
  ```bash
  nano .env
  ```
- Add the conent in .env file. To get the .env file click [here](https://github.com/effaaykhan/Graylog-Docker/blob/main/.env)
- In the .env file we need to add GRAYLO_PASSWORD_SECRET & GRAYLOG_ROOT_PASSWORD_SHA. Without them graylog will not start.
  - GRAYLOG_PASSWORD_SECRET
    ```bash
    < /dev/urandom tr -dc A-Z-a-z-0-9 | head -c${1:-96};echo;
    ```
  - GRAYLOG_ROOT_PASSWORD_SHA
    ```bash
    echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
    ```
- After the setup run the following command:
  ```bash
  docker-compose up -d
  ```

## Extractors
- [Regular Expression](https://github.com/effaaykhan/Graylog-Docker/blob/main/Extractors/Regex%20Extractor) extractor for fortigate logs

## Geo-Location 
- Step 1: Downlaod .mmdb databases from [MaxMind](https://www.maxmind.com/en/accounts/1130460/geoip/downloads).
  - GeoLite2-ASN.mmdb
    ```bash
    wget "https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-ASN&license_key=YOUR_LICENSE_KEY&suffix=tar.gz" -O GeoLite2-ASN.tar.gz
    ```
  - GeoLite2-City.mmdb
    ```bash
    wget "https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-City&license_key=YOUR_LICENSE_KEY&suffix=tar.gz" -O GeoLite2-City.tar.gz

    ```
- Step 2: Configure and Enable the geo-location processor under System>Configuration>Plugins and put the correct path of the databases
- Step 3: Create a Lookup Table
    - Data Adapter: Goto System>Lookup Tables>Data Adapters>Create Data Adapter> Geo IP MaxMind and fill the details. Select Type: City Database
    - Lookup Table: Now create lookup table and add the adapter
 
- Step 4: Create a new [Pipeline Rule](https://github.com/effaaykhan/Graylog-Docker/blob/main/Pipelines/geo-location)



# Forward LLM enriched logs to graylog.
- Step 1: Create a python file in ```/var/ossec/``` on wazuh-server:
  ```
  nano /var/ossec/yara_to_graylog.py
  ```
- Step 2: Paste the [yara_to_graylog.py](https://github.com/effaaykhan/Graylog-Docker/blob/main/yara_to_graylog.py) in the newly created file.
- Step 3: Now create a service in ```/etc/systemd/system/```
  ```
  sudo nano /etc/systemd/system/yara-graylog.service
  ```
- Step 4: Paste the [yara-graylog.service](https://github.com/effaaykhan/Graylog-Docker/blob/main/yara-graylog.service) in the newly created file.
