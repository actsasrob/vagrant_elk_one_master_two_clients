# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version ">= 1.5.2"

$ubuntu_setupscript = <<END
  echo 'setting up Ubuntu 16.10 yakkety yak...'
  echo 'installing Puppet 4.x...'
  wget https://apt.puppetlabs.com/puppetlabs-release-pc1-yakkety.deb
  sudo dpkg -i puppetlabs-release-pc1-yakkety.deb
  sudo apt-get update
  sudo apt-get -y install puppet-agent
 
  sudo grep secure_path /etc/sudoers \
  | sed -e 's#"$#:/opt/puppetlabs/bin"#' \
  | sudo tee /etc/sudoers.d/puppet-securepath

  echo ". /etc/profile.d/puppet-agent.sh" >> ~/.bashrc

  sudo apt-get -y install tree

  echo "Checking out aws-system-administration-resources git project..."
  git clone https://github.com/actsasrob/aws-system-administration-resources.git

  echo 'done.'
END

$ubuntu_setupscript_example_5_1 = <<END
  echo 'setting up Ubuntu 16.10 yakkety yak...'
  #sudo apt-get update

  sudo apt-get -y install tree

  echo "Checking out aws-system-administration-resources git project..."
  git clone https://github.com/actsasrob/aws-system-administration-resources.git

  cd aws-system-administration-resources/ch05/example_5-1
  ./setup_mezzanine.sh
 
  echo 'done.'
END

$ubuntu_setupscript_example_5_8 = <<END
  echo 'setting up Ubuntu 16.10 yakkety yak...'
  echo 'installing Puppet 4.x...'
  wget https://apt.puppetlabs.com/puppetlabs-release-pc1-yakkety.deb
  sudo dpkg -i puppetlabs-release-pc1-yakkety.deb
  sudo apt-get update
  sudo apt-get -y install puppet-agent

  sudo grep secure_path /etc/sudoers \
  | sed -e 's#"$#:/opt/puppetlabs/bin"#' \
  | sudo tee /etc/sudoers.d/puppet-securepath

  echo ". /etc/profile.d/puppet-agent.sh" >> ~/.bashrc
  source ~/.bashrc

  sudo apt-get -y install tree

  echo "Checking out aws-system-administration-resources git project..."
  git clone https://github.com/actsasrob/aws-system-administration-resources.git

  cd aws-system-administration-resources/ch05/example_5-8/myblog
  ./install_files.sh
  sudo puppet apply /etc/puppetlabs/code/environments/production/manifests/site_notec2.pp
 
  echo 'done.'
END

# Copy files into place
$centos_setupscript = <<END
  # Hardlock domain name
  echo 'supercede domain-name "example.com";' > /etc/dhcp/dhclient.conf

  # Install etc/hosts for convenience
  # cp /vagrant/etc-puppet/hosts /etc/hosts

  # Add /opt/puppetlabs to the sudo secure_path
  echo "Adding /opt/puppetlabs/bin to secure_path in /etc/sudoers..."
  sed -i -e 's#\(secure_path = .*\)$#\1:/opt/puppetlabs/bin#' /etc/sudoers

  # Install puppet.conf in user directory to share code directory
  #mkdir -p /home/vagrant/.puppetlabs/etc/puppet
  #cp /vagrant/etc-puppet/personal-puppet.conf /home/vagrant/.puppetlabs/etc/puppet/puppet.conf
  #chown -R vagrant:vagrant /home/vagrant/.puppetlabs

  # Install example hiera settings in global directory
  #mkdir -p /etc/puppetlabs/puppet
  #cp /vagrant/etc-puppet/puppet.conf /etc/puppetlabs/puppet/
  #mkdir -p /etc/puppetlabs/code
  #chown -R vagrant:vagrant /etc/puppetlabs

  # Provide the URL to the Puppet Labs yum repo on login
  # echo "Installaing puppet-agent..."
  # sudo yum install -y http://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
  # sudo yum install -y puppet-agent

  # Enable MotD
  sed -i -e 's/^PrintMotd no/PrintMotd yes/' /etc/ssh/sshd_config
  systemctl reload sshd
END

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  #config.vm.box = "puppetlabs/centos-7.2-64-nocm"
  #config.vm.box = "centos/7"
  #config.vm.box = "ubuntu/xenial64"
  config.vm.box = "ubuntu/yakkety64"
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
  end

  # clients
  config.vm.define "ubuntu_client", primary: true do |client|
    client.vm.hostname = "client.example.com"
    client.vm.network :private_network, ip: "192.168.250.10"
    client.vm.provision "shell", inline: $ubuntu_setupscript
  end

  config.vm.define "example_5_1", primary: true do |client|
    client.vm.hostname = "client.example.com"
    client.vm.network :private_network, ip: "192.168.250.11"
    client.vm.provision "shell", inline: $ubuntu_setupscript_example_5_1
  end

  config.vm.define "example_5_8", primary: true do |client|
    client.vm.hostname = "client.example.com"
    client.vm.network :private_network, ip: "192.168.250.11"
    client.vm.provision "shell", inline: $ubuntu_setupscript_example_5_8
  end

  config.vm.define "client", primary: true do |client|
    client.vm.hostname = "client.example.com"
    client.vm.network :private_network, ip: "192.168.250.10"
    client.vm.provision "shell", inline: $centos_setupscript
  end

  config.vm.define "web1", primary: true do |webserver|
    webserver.vm.hostname = "web1.example.com"
    webserver.vm.network :private_network, ip: "192.168.250.21"
    webserver.vm.provision "shell", inline: $centos_setupscript
  end
  config.vm.define "web2", primary: true do |webserver|
    webserver.vm.hostname = "web2.example.com"
    webserver.vm.network :private_network, ip: "192.168.250.22"
    webserver.vm.provision "shell", inline: $centos_setupscript
  end
  config.vm.define "web3", primary: true do |webserver|
    webserver.vm.hostname = "web3.example.com"
    webserver.vm.network :private_network, ip: "192.168.250.23"
    webserver.vm.provision "shell", inline: $centos_setupscript
  end

  # A puppetmaster
  config.vm.define "puppetmaster", autostart: false do |puppetmaster|
    puppetmaster.vm.hostname = "puppetmaster.example.com"
    puppetmaster.vm.network :private_network, ip: "192.168.250.5"
    puppetmaster.vm.provision "shell", inline: $centos_setupscript
  end

  # Puppet Server
  config.vm.define "puppetserver", autostart: false do |puppetserver|
    puppetserver.vm.hostname = "puppetserver.example.com"
    puppetserver.vm.network :private_network, ip: "192.168.250.6"
    puppetserver.vm.provision "shell", inline: $centos_setupscript
    puppetserver.vm.provider :virtualbox do |ps|
      ps.memory = 1024
    end
  end

  # Puppet Dashboard
  config.vm.define "dashboard", autostart: false do |puppetserver|
    puppetserver.vm.hostname = "dashserver.example.com"
    puppetserver.vm.network :private_network, ip: "192.168.250.7"
    puppetserver.vm.provision "shell", inline: $centos_setupscript
    puppetserver.vm.provider :virtualbox do |ps|
      ps.memory = 1024
    end
  end
end
