# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version ">= 1.5.2"

$ubuntu_client_setupscript = <<END
  set -x
  # Credit to: https://www.howtoforge.com/tutorial/how-to-install-elastic-stack-on-centos-7/

  # Hardlock domain name
  echo 'supercede domain-name "co";' > /etc/dhcp/dhclient.conf

  apt-get install -y wget net-tools

  mkdir -p /etc/pki/tls/certs/
  cp /vagrant/logstash-forwarder.crt /etc/pki/tls/certs/
  chmod 644 /etc/pki/tls/certs/logstash-forwarder.crt
  chown root:root /etc/pki/tls/certs/logstash-forwarder.crt

  echo "info: Install filebeat version ${FILEBEATVERSION}..."

  if [ ! -f /etc/filebeat/filebeat.yml ]; then
     wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
     wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-${FILEBEATVERSION}-amd64.deb
     dpkg -i filebeat-${FILEBEATVERSION}-amd64.deb
     if [ ! -f /etc/filebeat/filebeat.yml ]; then
        echo "error: filebeat not installed"
        exit 1
     fi
  else
     echo "info: filebeat already installed."
  fi

  grep '/var/log/syslog' /etc/filebeat/filebeat.yml > /dev/null 2>&1
  if [ "$?" -ne 0 ]; then
    sed -i '/paths:/a\\
    - /var/log/auth.log\\
    - /var/log/syslog' /etc/filebeat/filebeat.yml 
  fi

  sed -i '/var.log.\\*.log/d' /etc/filebeat/filebeat.yml

  sed -i 's/^output.elasticsearch:/#output.elasticsearch:/' /etc/filebeat/filebeat.yml
  sed -i 's/^\\([[:space:]]*hosts: \\["localhost:9200"]\\)/#\\1/' /etc/filebeat/filebeat.yml

  grep "^output.logstash:" /etc/filebeat/filebeat.yml > /dev/null 2>&1
  if [ "$?" -ne 0 ]; then
  sed -i '/^#output.logstash:.*/i\\
output.logstash:\\
  # The Logstash hosts\\
  hosts: ["192.168.250.21:5443"]\\
  bulk_max_size: 1024\\
  ssl.certificate_authorities: ["/etc/pki/tls/certs/logstash-forwarder.crt"]\\
  template.name: "filebeat"\\
  template.path: "filebeat.template.json"\\
  template.overwrite: false' /etc/filebeat/filebeat.yml
  fi

  systemctl enable filebeat
  systemctl start filebeat
END

# Copy files into place
$centos_client_setupscript = <<END
  set -x
  # Credit to: https://www.howtoforge.com/tutorial/how-to-install-elastic-stack-on-centos-7/

  # Hardlock domain name
  echo 'supercede domain-name "co";' > /etc/dhcp/dhclient.conf

  yum install -y wget net-tools

  mkdir -p /etc/pki/tls/certs/
  cp /vagrant/logstash-forwarder.crt /etc/pki/tls/certs/
  chmod 644 /etc/pki/tls/certs/logstash-forwarder.crt
  chown root:root /etc/pki/tls/certs/logstash-forwarder.crt

  echo "info: Install filebeat version ${FILEBEATVERSION}..."

  if [ ! -f /etc/filebeat/filebeat.yml ]; then
     rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
     wget --quiet https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-${FILEBEATVERSION}-x86_64.rpm
     rpm -ivh filebeat-${FILEBEATVERSION}-x86_64.rpm
     if [ ! -f /etc/filebeat/filebeat.yml ]; then
        echo "error: filebeat not installed"
	exit 1
     fi
  else
     echo "info: filebeat already installed."
  fi

  grep '/var/log/secure' /etc/filebeat/filebeat.yml > /dev/null 2>&1
  if [ "$?" -ne 0 ]; then
    sed -i '/^  paths:.*$/a\\
    - /var/log/secure\\
    - /var/log/messages' /etc/filebeat/filebeat.yml
  fi

  sed -i '/var.log.\\*.log/d' /etc/filebeat/filebeat.yml

  sed -i 's/^output.elasticsearch:/#output.elasticsearch:/' /etc/filebeat/filebeat.yml
  sed -i 's/^\\([[:space:]]*hosts: \\["localhost:9200"]\\)/#\\1/' /etc/filebeat/filebeat.yml

  grep "^output.logstash:" /etc/filebeat/filebeat.yml > /dev/null 2>&1
  if [ "$?" -ne 0 ]; then
  sed -i '/^#output.logstash:.*/i\\
output.logstash:\\
  # The Logstash hosts\\
  hosts: ["192.168.250.21:5443"]\\
  bulk_max_size: 1024\\
  ssl.certificate_authorities: ["/etc/pki/tls/certs/logstash-forwarder.crt"]\\
  template.name: "filebeat"\\
  template.path: "filebeat.template.json"\\
  template.overwrite: false' /etc/filebeat/filebeat.yml
  fi

  systemctl enable filebeat
  systemctl start filebeat
END

$centos_master_setupscript = <<END
  # Credit to: https://www.howtoforge.com/tutorial/how-to-install-elastic-stack-on-centos-7/


  # Install for VBoxGuestAdditons
  #yum -y install epel-release
  #yum install -y gcc kernel-devel kernel-headers dkms make bzip2 perl
  #export KERN_DIR=/usr/src/kernels/`uname -r`/build

  # Hardlock domain name
  echo 'supercede domain-name "co";' > /etc/dhcp/dhclient.conf

  echo "info: disable SELinux"
  sed -i -e 's/^SELINUX=.*/SELINUX=disabled/' /etc/sysconfig/selinux 
  setenforce 0

  echo "Check SELinux status..."
  getenforce

  # Note: In order to install Oracle Java 8 JDK, you will need to go to the Oracle Java 8 JDK Downloads Page, accept the license agreement, and copy the download link of the appropriate Linux .rpm package. Substitute the copied download link in place of the highlighted part of the wget command.

  yum install -y wget net-tools

  ORACLE_JDK_VERSION=8u171
  ORACLE_JDK_BUILD=b11
  echo "info: Install Oracle jdk 8"
  which java > /dev/null 2>&1
  if [ "$?" -ne 0 ]; then
     wget --quiet --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http:%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/${ORACLE_JDK_VERSION}-${ORACLE_JDK_BUILD}/512cd62ec5174c3487ac17c61aaa89e8/jdk-${ORACLE_JDK_VERSION}-linux-x64.rpm"
     rpm -ivh jdk-8u171-linux-x64.rpm

     which java > /dev/null 2>&1
     if [ "$?" -ne 0 ]; then
        echo "error: java not installed"
        exit 1
     fi
  else
     echo "info: Oracle JDK already intalled."
  fi
  java -version

  echo "info: Install elasticsearch version ${ELASTICSEARCHVERSION}..."

  if [ ! -f /usr/share/elasticsearch/bin/elasticsearch ]; then
     rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
     wget --quiet https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-${ELASTICSEARCHVERSION}.rpm
     rpm -ivh elasticsearch-${ELASTICSEARCHVERSION}.rpm
     if [ ! -f /usr/share/elasticsearch/bin/elasticsearch ]; then
        echo "error: elasticsearch not installed"
	exit 1
     fi
  else
     echo "info: elasticsearch already installed."
  fi

  sed -i -e 's/^#bootstrap.memory_lock:.*/bootstrap.memory_lock: true/' /etc/elasticsearch/elasticsearch.yml
  sed -i -e 's/^#network.host:.*/network.host: localhost/' /etc/elasticsearch/elasticsearch.yml
  sed -i -e 's/^#http.port:.*/http.port: 9200/' /etc/elasticsearch/elasticsearch.yml

  sed -i -e 's/^#LimitMEMLOCK=.*/LimitMEMLOCK=infinity/' /usr/lib/systemd/system/elasticsearch.service

  touch /etc/sysconfig/elasticsearch
  chown root:elasticsearch /etc/sysconfig/elasticsearch
  grep "MAX_LOCKED_MEMORY=" /etc/sysconfig/elasticsearch > /dev/null 2>&1
  if [ "$?" -eq 0 ]; then
     sed -i -e 's/^#MAX_LOCKED_MEMORY=.*/MAX_LOCKED_MEMORY=unlimited/' /etc/sysconfig/elasticsearch
  else
     echo "MAX_LOCKED_MEMORY=unlimited" >> /etc/sysconfig/elasticsearch
  fi

  systemctl daemon-reload
  systemctl enable elasticsearch
  systemctl start elasticsearch

  sleep 5
  port_9200_listen=$(netstat -plntu | grep -i listen | grep 9200 | wc -l)
  if [ "$port_9200_listen" -eq 0 ]; then
     echo "error: elasticsearch not listening on port 9200"
     #exit 1
  fi

  curl -XGET 'localhost:9200/_nodes?filter_path=**.mlockall&pretty'
  if [ "$?" -ne 0 ]; then
     echo "error: elasticsearch mlockall not set to true"
     #exit 1
  fi

  echo
  echo "info: Install and Configure Kibana with Nginx"
  if [ ! -f /etc/kibana/kibana.yml ]; then  
     wget --quiet https://artifacts.elastic.co/downloads/kibana/kibana-${KIBANAVERSION}-x86_64.rpm
     rpm -ivh kibana-${KIBANAVERSION}-x86_64.rpm
  else
     echo "info: kibana already installed"
  fi
  sed -i -e 's/^#server.port:.*/server.port: 5601/' /etc/kibana/kibana.yml
  sed -i -e 's/^#server.host:.*/server.host: "localhost"/' /etc/kibana/kibana.yml
  sed -i -e 's|^#elasticsearch.url:.*|elasticsearch.url: "http://localhost:9200"|' /etc/kibana/kibana.yml 

  systemctl enable kibana
  systemctl start kibana

  sleep 3
  kibana_listeners=$(netstat -plntu | grep -i listen | grep 5601 | wc -l)
  if [ "$kibana_listeners" -eq 0 ]; then
     echo "error: no kibana listeners on port 5601"
     exit 1
  fi

  yum -y install epel-release
  yum -y install nginx httpd-tools

  # http://fahdshariff.blogspot.com/2012/12/sed-mutli-line-replacement-between-two.html
  #set -x 
  sed -i '/^[[:space:]]*server [{].*/,/^}$/ {/^[[:space:]]*server [{].*/n;/^}$/!d}' /etc/nginx/nginx.conf
  sed -i '/^[[:space:]]*server [{].*/d' /etc/nginx/nginx.conf

  cat <<'EOF' > /etc/nginx/conf.d/kibana.conf
server {
    listen 80;
 
    server_name elkstack.co;
 
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.kibana-user;
 
    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
EOF

  echo "Password1" | htpasswd -i -c /etc/nginx/.kibana-user admin

  nginxout=$(nginx -t 2>&1) 
  echo $nginxout | grep "test is successful" > /dev/null 2>&1
  if [ "$?" -ne 0 ]; then
     echo "error: nginx configuration error. run 'sudo nginx -t' to troubleshoot"
  fi

  systemctl enable nginx
  systemctl start nginx

  echo "info:  Install and Configure Logstash"
  if [ ! -f /etc/logstash/logstash.yml ]; then
     wget --quiet https://artifacts.elastic.co/downloads/logstash/logstash-${LOGSTASHVERSION}.rpm
     rpm -ivh logstash-${LOGSTASHVERSION}.rpm
  else
     echo "info: logstash already installed"
  fi

  grep "^# Server IP Address" /etc/pki/tls/openssl.cnf > /dev/null 2>&1
  if [ "$?" -ne 0 ]; then
     sed -i '/ v3_ca ]/a\\
# Server IP Address\\
subjectAltName = IP: 192.168.250.21' /etc/pki/tls/openssl.cnf
  fi

  if [ ! -f /etc/pki/tls/certs/logstash-forwarder.crt ]; then
     openssl req -config /etc/pki/tls/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout /etc/pki/tls/private/logstash-forwarder.key -out /etc/pki/tls/certs/logstash-forwarder.crt
  fi

  cp -f /etc/pki/tls/certs/logstash-forwarder.crt /vagrant/

  cat << 'EOF' > /etc/logstash/conf.d/filebeat-input.conf 
input {
  beats {
    port => 5443
    ssl => true
    ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
    ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
  }
}
EOF

  cat << 'EOF' > /etc/logstash/conf.d/syslog-filter.conf
filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}
EOF

  cat << 'EOF' > /etc/logstash/conf.d/output-elasticsearch.conf
output {
  elasticsearch { hosts => ["localhost:9200"]
    hosts => "localhost:9200"
    manage_template => false
    index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
    document_type => "%{[@metadata][type]}"
  }
}
EOF

  systemctl enable logstash
  systemctl start logstash

  sleep 8
  logstash_listeners=$(netstat -plntu | grep -i listen | grep 5443 | wc -l)
  if [ "$logstash_listeners" -eq 0 ]; then
     echo "error: no logstash listeners on port 5443"
  fi

  sed -i -e 's/^PrintMotd no/PrintMotd yes/' /etc/ssh/sshd_config
  systemctl reload sshd
  whoami
END

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # NOTE: Before running 'vagrant up ...' you must first install the vagrant-sshfs plugin and install an sftp-server. eg.
  #       sudo vagrant plugin install vagrant-sshfs
  #       ubuntu: sudo apt-get install openssh-sftp-server
  #       rhel:   sudo yum install openssh-sftp-server
  #
  config.vm.synced_folder ".", "/vagrant", type: "sshfs"
  config.vm.box = "centos/7"
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
  end

  # Configure the master 
  config.vm.define "master", primary: true do |master|
    master.vm.box = "centos/7"
    master.vm.hostname = "elkstack.co"
    master.vm.network :private_network, ip: "192.168.250.21"
    master.vm.provider :virtualbox do |vb|
       vb.memory = 4096
       vb.cpus = 2
    end
    master.vm.provision "shell", inline: $centos_master_setupscript, env: {"FILEBEATVERSION"=>"6.2.1", "ELASTICSEARCHVERSION"=>"6.2.1", "LOGSTASHVERSION"=>"6.2.1", "KIBANAVERSION"=>"6.2.1"}

  end

  # Configure the CentOS7 client
  config.vm.define "client1", primary: true do |client1|
    client1.vm.box = "centos/7"
    client1.vm.hostname = "client1.co"
    client1.vm.network :private_network, ip: "192.168.250.22"
    client1.vm.provider :virtualbox do |vb|
       vb.memory = 1024
       vb.cpus = 2
    end
    client1.vm.provision "shell", inline: $centos_client_setupscript, env: {"FILEBEATVERSION"=>"6.2.1", "ELASTICSEARCHVERSION"=>"6.2.1", "LOGSTASHVERSION"=>"6.2.1", "KIBANAVERSION"=>"6.2.1"}

  end

  # Configure the Ubuntu 16.04 client
  config.vm.define "client2", primary: true do |client2|
    client2.vm.box = "ubuntu/xenial64"
    client2.vm.hostname = "client2.co"
    client2.vm.network :private_network, ip: "192.168.250.23"
    client2.vm.provider :virtualbox do |vb|
       vb.memory = 1024
       vb.cpus = 2
    end
    client2.vm.provision "shell", inline: $ubuntu_client_setupscript, env: {"FILEBEATVERSION"=>"6.2.1", "ELASTICSEARCHVERSION"=>"6.2.1", "LOGSTASHVERSION"=>"6.2.1", "KIBANAVERSION"=>"6.2.1"}
  end

end
