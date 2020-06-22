# prometheus-stack
Prometheus Grafana alertmanager pushgateway exporters

## Installation Steps:
Prometheus Installation
------------------------
sudo useradd -M -r -s /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus
Download and extract the pre-compiled binaries:

wget https://github.com/prometheus/prometheus/releases/download/v2.16.0/prometheus-2.16.0.linux-amd64.tar.gz
tar xzf prometheus-2.16.0.linux-amd64.tar.gz prometheus-2.16.0.linux-amd64/
Move the files from the downloaded archive to the appropriate locations and set ownership:

sudo cp prometheus-2.16.0.linux-amd64/{prometheus,promtool} /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}
sudo cp -r prometheus-2.16.0.linux-amd64/{consoles,console_libraries} /etc/prometheus/
sudo cp prometheus-2.16.0.linux-amd64/prometheus.yml /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
Briefly test your setup by running Prometheus in the foreground:

prometheus --config.file=/etc/prometheus/prometheus.yml
Create a systemd unit file for Prometheus:

sudo vi /etc/systemd/system/prometheus.service
Define the Prometheus service in the unit file:

[Unit]
Description=Prometheus Time Series Collection and Processing Server
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
Start and enable the Prometheus service:

sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
Make an HTTP request to Prometheus to verify it is able to respond:

curl localhost:9090



Configuration:
--------------
Log in to your Prometheus server.

Edit the Prometheus configuration file:

sudo vi /etc/prometheus/prometheus.yml
Locate the global.scrape_interval setting and change it to 10s:

global:
  scrape_interval: 10s
Reload the Prometheus configuration:

sudo killall -HUP prometheus
Another way to reload the configuration is to simply restart Prometheus (you do not need to do this if you used the killall -HUP method):

sudo systemctl restart prometheus
Query the Prometheus API to verify your changes took effect:

curl localhost:9090/api/v1/status/config
You should see global:\n scrape_interval: 10s in the output.


Node Exporter:
---------------
Log in to the new server. We will configure this new server for Prometheus monitoring using Node Exporter.

Create a user for Node Exporter:

sudo useradd -M -r -s /bin/false node_exporter
Download and extract the Node Exporter binary:

wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
tar xvfz node_exporter-0.18.1.linux-amd64.tar.gz
Copy the Node Exporter binary to the appropriate location and set ownership:

sudo cp node_exporter-0.18.1.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
Create a systemd unit file for Node Exporter:

sudo vi /etc/systemd/system/node_exporter.service
Define the Node Exporter service in the unit file:

[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
Start and enable the node_exporter service:

sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
You can retrieve the metrics served by Node Exporter like so:

curl localhost:9100/metrics
Configure Prometheus to Scrape Metrics

Log in to your Prometheus server and configure Prometheus to scrape metrics from the new server.

Edit the Prometheus config file:

sudo vi /etc/prometheus/prometheus.yml
Locate the scrape_configs section and add a new entry under that section. You will need to supply the private IP address of your new playground server for targets.

...

- job_name: 'Linux Server'
  static_configs:
  - targets: ['<PRIVATE_IP_ADDRESS_OF_NEW_SERVER>:9100']

...
Reload the Prometheus config:

sudo killall -HUP prometheus
Navigate to the Prometheus expression browser in your web browser using the public IP address of your Prometheus server: <PROMETHEUS_SERVER_PUBLIC_IP>:9090.

Run some queries to retrieve metric data about your new server:

up
node_filesystem_avail_bytes



