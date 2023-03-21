Ref https://sbcode.net/grafana/install-loki-service/

# Install Loki Binary and Start as a Service
## To check the latest version of Grafana Loki, visit the Loki releases page. https://github.com/grafana/loki/releases/
```
cd /usr/local/bin
curl -O -L "https://github.com/grafana/loki/releases/download/v2.4.1/loki-linux-amd64.zip"
unzip "loki-linux-amd64.zip"
```
### And allow the execute permission on the Loki binary
```
chmod a+x "loki-linux-amd64"
```

### Create the Loki config
### Now create the Loki config file.
```
sudo nano config-loki.yml
```
### And add this text.
```
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093

# By default, Loki will send anonymous, but uniquely-identifiable usage and configuration
# analytics to Grafana Labs. These statistics are sent to https://stats.grafana.org/
#
# Statistics help us better understand how Loki is used, and they show us performance
# levels for most users. This helps us prioritize features and documentation.
# For more information on what's sent, look at
# https://github.com/grafana/loki/blob/main/pkg/usagestats/stats.go
# Refer to the buildReport method to see what goes into a report.
#
# If you would like to disable reporting, uncomment the following lines:
#analytics:
#  reporting_enabled: false
```

### Note:- This default configuration was copied from https://raw.githubusercontent.com/grafana/loki/master/cmd/loki/loki-local-config.yaml.

## Configure Loki to run as a service

### Create a user specifically for the Loki service

```
sudo useradd --system loki
```
### Create a file called loki.service
```
sudo nano /etc/systemd/system/loki.service
```
### Add the script and save
```
[Unit]
Description=Loki service
After=network.target

[Service]
Type=simple
User=loki
ExecStart=/usr/local/bin/loki-linux-amd64 -config.file /usr/local/bin/config-loki.yml

[Install]
WantedBy=multi-user.target
```

### Now start and check the service is running.
```
sudo service loki start
sudo service loki status
```
### We can now leave the new Loki service running.

### If you ever need to stop the new Loki service, then type
```
sudo service loki stop
sudo service loki status
```

Create Data_Source in Grafana
URL http://127.0.0.1:3100
click on Save and test


*************************************
# Install Promtail service to read logs file
### To check the latest version of Promtail, visit the Loki releases page. https://github.com/grafana/loki/releases/

```
cd /usr/local/bin

curl -O -L "https://github.com/grafana/loki/releases/download/v2.4.1/promtail-linux-amd64.zip"

unzip "promtail-linux-amd64.zip"
```
And allow the execute permission on the Promtail binary


sudo chmod a+x "promtail-linux-amd64"
Create the Promtail config
Now we will create the Promtail config file.


sudo nano config-promtail.yml
And add this script,


server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: 'http://localhost:3100/loki/api/v1/push'

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*log
This default configuration was copied from https://raw.githubusercontent.com/grafana/loki/master/clients/cmd/promtail/promtail-local-config.yaml when it was version 2.4.1. There may be changes to this config depending on any future updates to Loki.

Configure Promtail as a Service
Now we will configure Promtail as a service so that we can keep it running in the background.

Create user specifically for the Promtail service


sudo useradd --system promtail
Create a file called promtail.service


sudo nano /etc/systemd/system/promtail.service
And add this script,


[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=promtail
ExecStart=/usr/local/bin/promtail-linux-amd64 -config.file /usr/local/bin/config-promtail.yml

[Install]
WantedBy=multi-user.target
Now start and check the service is running.


sudo service promtail start
sudo service promtail status
We can now leave the new Promtail service running.

Now, since this example uses Promtail to read system log files, the promtail user won't yet have permissions to read them.

So add the user promtail to the adm group


usermod -a -G adm promtail
Verify that the user is now in the adm group


id promtail
Restart Promtail and check status


sudo service promtail restart
sudo service promtail status
If you ever need to stop the new Promtail service, then type


sudo service promtail stop
sudo service promtail status