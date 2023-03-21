sudo apt update

Get the latest from Link
https://grafana.com/grafana/download?edition=oss

sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_9.2.5_amd64.deb
sudo dpkg -i grafana_9.2.5_amd64.deb


sudo systemctl start grafana-server
sudo systemctl status grafana-server

for login
url:3000
admin
admin
