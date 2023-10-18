Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.ssh.forward_agent = true
  config.ssh.forward_x11 = true

  config.vm.provider :virtualbox do |vb|
    vb.memory = 4096
  end

  config.vm.provision :shell, name: "Setup-Puppet", privileged: false, inline: <<-SHELL
    sudo yum install -y git wget
    RPM_FILE=puppetlabs-release-el7.noarch.rpm
    wget "http://yum.puppetlabs.com/${RPM_FILE}" 2> /dev/null
    rpm -i ${RPM_FILE} 
    sudo yum install -y centos-release-scl-rh centos-release-scl
    sudo yum install -y --enablerepo=centos-sclo-rh -y install rh-ruby30 rh-ruby30-rubygems rh-ruby30-ruby-devel
    sudo ln -s /opt/rh/rh-ruby30/root/usr/bin/* /usr/bin
    sudo ln -s /opt/rh/rh-ruby30/root/usr/lib64/libruby.so* /usr/lib64
    sudo gem install puppet
    sudo gem install librarian-puppet
    sudo ln -s /opt/rh/rh-ruby30/root/usr/local/bin/puppet /usr/bin
    sudo ln -s /opt/rh/rh-ruby30/root/usr/local/bin/librarian-puppet /usr/bin
  SHELL

  config.vm.synced_folder ".","/vagrant", type: "rsync"
  config.vm.synced_folder "puppet/environments","/tmp/vagrant-puppet/environments", type: "rsync"

  config.vm.provision :shell, name: "Puppet-librarian", privileged: false, inline: <<-SHELL
    cd /vagrant/puppet/environments/default/
    if [ ! -e Puppetfile.lock -o Puppetfile.lock -ot Puppetfile ]; then
      chmod -R a+rw modules
      librarian-puppet install
      [ $? -eq 0 ] && touch Puppetfile.lock
    else
      echo "Puppetfile is locked"
    fi
  SHELL
  
  config.vm.provision :puppet do |puppet|
    puppet.environment_path = "puppet/environments"
    puppet.environment = "default"
    puppet.hiera_config_path = "puppet/environments/default/hiera/hiera.yaml"
    puppet.facter = {
      #"user" => "#{ENV['USER']}",
      "puppet_dir" => "/vagrant/puppet",
    }
  end

end
