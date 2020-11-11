# DIPET Apache Flink  testbed cluster
# beta v0.0.1
# Bootstrap script for Prometheus cluster
#
#Copyright 2019, West University of Timisoara, Timisoara, Romania
#    http://www.uvt.ro/
#Developers:
# * Gabriel Iuhasz, iuhasz.gabriel@info.uvt.ro
#
#Licensed under the Apache License, Version 2.0 (the "License");
#you may not use this file except in compliance with the License.
#You may obtain a copy of the License at:
#    http://www.apache.org/licenses/LICENSE-2.0

# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

$master_script = <<SCRIPT
#!/bin/bash
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
# add-apt-repository ppa:jonathonf/python-3.6 -y
apt-get update
apt-get install collectd -y
apt-get install wget -y
apt-get install python3-pip -y

sudo chown vagrant:vagrant -R /opt
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.13.0-rc.0/prometheus-2.13.0-rc.0.linux-amd64.tar.gz
tar -xzvf prometheus-2.13.0-rc.0.linux-amd64.tar.gz && cd /opt/prometheus-2.13.0-rc.0.linux-amd64

cat <<EOF > /opt/prometheus-2.13.0-rc.0.linux-amd64/prometheus.yml
global:
  scrape_interval:     1s # By default, scrape targets every 15 seconds.
  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'dipet-monitor'
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'node-prometheus'
    static_configs:
      - targets: ['10.211.55.101:9100']
  - job_name: 'flink'
    static_configs:
      - targets: ['10.211.55.101:9249']
EOF
cd /opt/prometheus-2.13.0-rc.0.linux-amd64
# Start prometheus
# nohup ./prometheus > prometheus.log 2>&1 &

# Register node_exporter as service
sudo mkdir /usr/lib/systemd/system
sudo cp /vagrant/prometheus.service /usr/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable prometheus.service
sudo systemctl start prometheus.service
# sudo service prometheus.service start

cd /opt

# Download grafana
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_4.2.0_amd64.deb -O /opt/grafana_4.2.0_amd64.deb

sudo apt-get install -y adduser libfontconfig

# Install grafana
sudo dpkg -i grafana_4.2.0_amd64.deb

# Start grafana service
sudo service grafana-server start

# Run on every boot
sudo update-rc.d grafana-server defaults



# Set Swappiness value to 10 instead of 60
sysctl -w vm.swappiness=10
cat /proc/sys/vm/swappiness

cat > /etc/hosts <<EOF
127.0.0.1       localhost
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
EOF


sudo chown vagrant:vagrant -R /opt
SCRIPT


$hosts_script = <<SCRIPT
#!/bin/bash

export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
apt-get update
apt-get install wget -y
apt-get install collectd -y
apt-get install python3-pip -y

sudo chown vagrant:vagrant -R /opt

cd /opt
wget https://mirror.efect.ro/apache/flink/flink-1.11.2/flink-1.11.2-bin-scala_2.12.tgz
tar -xzvf flink-1.11.2-bin-scala_2.12.tgz && cd /opt/flink-1.11.2

# Configuration file
mv /opt/flink-1.11.2/conf/flink-conf.yaml /opt/flink-1.11.2/conf/flink-conf.yaml.b
cp /vagrant/flink-conf.yaml /opt/flink-1.11.2/conf/

# Register flink cluster as service
sudo mkdir /usr/lib/systemd/system
sudo cp /vagrant/flinkcluster.service /usr/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable flinkcluster.service
sudo systemctl start flinkcluster.service
#sudo service flinkcluster.service start

#Configuration file
mv /opt/flink-1.11.2/conf/flink-conf.yaml /opt/flink-1.11.2/conf/flink-conf.yaml.b
cp /vagrant/flink-conf.yaml /opt/flink-1.11.2/conf/

# cd /opt/flink-1.11.2/bin
# sudo ./start-cluster.sh

# Installing Prometheus node_exporter
cd /opt
wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz

# Extract node_exporter
tar -xvzf node_exporter-0.18.1.linux-amd64.tar.gz

sudo cp /vagrant/node_exporter.service /usr/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable node_exporter.service
sudo service node_exporter start

SCRIPT

Vagrant.configure("2") do |config|

  # Define base image
  config.vm.box = "ubuntu/bionic64"
  #config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  # Manage /etc/hosts on host and VMs
  config.hostmanager.enabled = false
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.hostmanager.ignore_private_ip = false

  #Set same Username and Password
  #config.ssh.username = "diper"
  #config.ssh.password = "dipet"

  #Path to Private SSH key location
  #config.ssh.private_key_path = "<path/to/key>"

  #SSH default host and port settings
  #config.ssh.host ="<host>"
  #config.ssh.port = "<port>"


  config.vm.define :master do |master|
    master.vm.provider :virtualbox do |v|
      v.name = "prometheusflink"
      v.customize ["modifyvm", :id, "--memory", "4096"]
    end
    master.disksize.size = '20GB'
    master.vm.network :private_network, ip: "10.211.55.100"
    master.vm.hostname = "prometheusflink"
    master.vm.network :forwarded_port, host:9090, guest: 9090
    master.vm.network :forwarded_port, guest: 22, host: 2185
    master.vm.provision :shell, :inline => $master_script
    master.vm.provision :hostmanager
  end

  config.vm.define :slave1 do |slave1|
    slave1.vm.box = "ubuntu/bionic64"
    slave1.vm.provider :virtualbox do |v|
      v.name = "flinknode1"
      v.customize ["modifyvm", :id, "--memory", "4096"]
    end
    slave1.vm.network :private_network, ip: "10.211.55.101"
    slave1.vm.hostname = "flinknode1"
    slave1.vm.network :forwarded_port, guest: 22, host: 2186
    slave1.vm.network :forwarded_port, guest: 8080, host: 8080
    slave1.vm.network :forwarded_port, guest: 8081, host: 8081
    slave1.vm.provision :shell, :inline => $hosts_script
    slave1.vm.provision :hostmanager
  end

end