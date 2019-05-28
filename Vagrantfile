require 'vagrant-hosts'
require 'vagrant-vbguest'

# Warning this will set no firewall rules!
# thingsboard REQUIRES Oracle Java, openjdk wont cut it. Get linux x64 tar.gz from: https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html
# This file gets you to about here: https://thingsboard.io/docs/user-guide/install/linux/#configure-thingsboard-to-use-the-external-database
# for user account details see: https://thingsboard.io/docs/samples/demo-account/
# Default system administrator account:
# login - sysadmin@thingsboard.org.
# password - sysadmin.


VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  $ubuntu_prepare = <<-SCRIPT
# Update the OS
apt-get -y update
# Remove the default jre as we are going to put on Oracle JRE for thingsboard
apt-get -y remove default-jre
# Ensure some quality of life stuff is installed
apt-get -y install screen vim htop

JRE_ARCHIVE_SOURCE_DIR=/vagrant/data
JRE_ARCHiVE_FILE=${JRE_ARCHIVE_SOURCE_DIR}/jre-8u212-linux-x64.tar.gz
JRE_VERSION=`tar -tf $JRE_ARCHiVE_FILE | egrep '^[^/]+/$' | head -c -2` 2>> /dev/null
JRE_DEST_DIR=/usr/local/java
JRE_LOCATION=${JRE_DEST_DIR}/${JRE_VERSION}

echo ${JRE_ARCHIVE_SOURCE}
echo ${JRE_ARCHiVE_FILE}
echo ${JRE_VERSION}
echo ${JRE_DEST_DIR}
echo ${JRE_LOCATION}

mkdir -p ${JRE_DEST_DIR}
tar zxvf ${JRE_ARCHiVE_FILE} -C ${JRE_DEST_DIR}

cat >> /etc/profile <<EOF
JAVA_HOME=${JRE_LOCATION}
PATH=$PATH:${JRE_DEST_DIR}/bin
export JAVA_HOME
export JRE_HOME
export PATH
EOF

# update alternatives
update-alternatives --install "/usr/bin/java" "java" "${JRE_LOCATION}/bin/java" 1 >> /dev/null
update-alternatives --set java ${JRE_LOCATION}/bin/java >> /dev/null

# defo remove openjdk
apt-get purge -q openjdk-\*

# Add cassandra repository
echo 'deb http://www.apache.org/dist/cassandra/debian 311x main' | tee --append /etc/apt/sources.list.d/cassandra.list > /dev/null
curl https://www.apache.org/dist/cassandra/KEYS | apt-key add -
apt-get -y update
## Cassandra installation
apt-get -y install cassandra
## Tools installation
apt-get -y install cassandra-tools

# Download the thingsboard deb
wget --quiet --output-document /tmp/thingsboard-2.3.1.deb https://github.com/thingsboard/thingsboard/releases/download/v2.3.1/thingsboard-2.3.1.deb
dpkg -i /tmp/thingsboard-2.3.1.deb

# Copy over our config with cassandra stuff in
/bin/cp -f /vagrant/config/thingsboard.yml /etc/thingsboard/conf/thingsboard.yml

# Run the thingsboard installer with demo content, and if successful, start the service
/usr/share/thingsboard/bin/install/install.sh --loadDemo && systemctl start thingsboard

SCRIPT


  # Define how much RAM and number of CPUs our VM(s) will have
  config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "4096"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
  end

  # Specifcally disable the default vagrant mapping to prevent accidental modifications
  config.vm.synced_folder "./data", "/vagrant/data", disabled: false
  config.vm.synced_folder "./config", "/vagrant/config", disabled: false

  # Define the vagrant name of our VM
  config.vm.define "experiments-thingsboard" do |host|

    # Define what the machines hostname will be
    host.vm.hostname = "thingsboard"
    # Define the source box - in this case the official CentOS 7 Vagrant box
    host.vm.box = "ubuntu/bionic64"
    
    # Host specific setting to not update Virtualbox tools
    # host.vbguest.auto_update = false
    
    # Define any network mappings
    host.vm.network "forwarded_port", guest: 8080, host: 8080, host_ip: "127.0.0.1", protocol: "tcp"
    # host.vm.network "forwarded_port", guest: 443, host: 8443, host_ip: "127.0.0.1", protocol: "tcp"

    # Tell vagrant to provision the box on first boot with the script we defined above
    host.vm.provision "shell", inline: $ubuntu_prepare, privileged: true
  end

  # Globally turn off auto-updating of VB Guest tools
  if Vagrant.has_plugin? "vagrant-vbguest"
	  config.vbguest.auto_update = false
    #	config.vbguest.no_remote   = true
    #	config.vbguest.no_install  = true
  end

end
