Make sure your EC2 instance:

Runs Amazon Linux 2 or Ubuntu 20.04+

Has port 3000 (Grafana) and 9090 (Prometheus) open in the Security Group

Has sudo privileges

Update the system:

sudo yum update -y     # Amazon Linux
# OR
sudo apt update -y     # Ubuntu

✅ 2. Install Prometheus
Create prometheus user
sudo useradd --no-create-home --shell /bin/false prometheus

Create directories
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

Download Prometheus

(Replace version if a newer one exists)

cd /tmp
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz
tar -xvf prometheus-2.54.1.linux-amd64.tar.gz
cd prometheus-2.54.1.linux-amd64

Move binaries
sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/

Move config & consoles
sudo mv consoles /etc/prometheus/
sudo mv console_libraries /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/

Set permissions
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus

📌 3. Create Prometheus systemd Service
sudo tee /etc/systemd/system/prometheus.service > /dev/null << EOF
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/

[Install]
WantedBy=multi-user.target
EOF


Reload systemd and start the service:

sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus


Check status:

sudo systemctl status prometheus


Prometheus runs at:
👉 http://<EC2-IP>:9090

✅ 4. Install Grafana
Add Grafana repo (for Amazon Linux / RHEL)
sudo tee /etc/yum.repos.d/grafana.repo > /dev/null << EOF
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
EOF


Install Grafana:

sudo yum install grafana -y
# OR for Ubuntu:
# sudo apt install -y apt-transport-https
# sudo apt install grafana -y


Enable & start Grafana:

sudo systemctl start grafana-server
sudo systemctl enable grafana-server


Access Grafana:
👉 http://<EC2-IP>:3000

Default credentials:

Username: admin
Password: admin


You will be prompted to set a new password.

🎯 5. Connect Prometheus to Grafana
In Grafana UI:

Go to Connections → Data Sources

Click Add data source

Choose Prometheus

Set URL:

http://localhost:9090


Click Save & Test

Should show "Data source is working".

📊 6. Create a Dashboard in Grafana
Option A: Import ready-made dashboard

In Grafana → Dashboards

Click Import

Use dashboard ID from Grafana.com, e.g.:

Node Exporter Full Dashboard (ID: 1860)

Prometheus Stats (ID: 3662)

Select your Prometheus data source

Click Import

✔ You now have a full dashboard!