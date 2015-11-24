# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
	# The most common configuration options are documented and commented below.
	# For a complete reference, please see the online documentation at
	# https://docs.vagrantup.com.

	# Every Vagrant development environment requires a box. You can search for
	# boxes at https://atlas.hashicorp.com/search.
	config.vm.box = "ubuntu/trusty32"

	# Disable automatic box update checking. If you disable this, then
	# boxes will only be checked for updates when the user runs
	# `vagrant box outdated`. This is not recommended.
	# config.vm.box_check_update = false

	# Create a forwarded port mapping which allows access to a specific port
	# within the machine from a port on the host machine. In the example below,
	# accessing "localhost:8080" will access port 80 on the guest machine.
	config.vm.network "forwarded_port", guest: 80, host: 8080
	config.vm.network "forwarded_port", guest: 3306, host: 3308

	# Create a private network, which allows host-only access to the machine
	# using a specific IP.
	config.vm.network "private_network", ip: "192.168.33.11"

	# Create a public network, which generally matched to bridged network.
	# Bridged networks make the machine appear as another physical device on
	# your network.
	# config.vm.network "public_network"

	# Share an additional folder to the guest VM. The first argument is
	# the path on the host to the actual folder. The second argument is
	# the path on the guest to mount the folder. And the optional third
	# argument is a set of non-required options.
	config.vm.synced_folder ".", "/vagrant", type: "nfs"

	# Provider-specific configuration so you can fine-tune various
	# backing providers for Vagrant. These expose provider-specific options.
	# Example for VirtualBox:
	#
	# config.vm.provider "virtualbox" do |vb|
	#	 # Display the VirtualBox GUI when booting the machine
	#	 vb.gui = true
	#
	#	 # Customize the amount of memory on the VM:
	#	 vb.memory = "1024"
	# end
	#
	# View the documentation for the provider you are using for more
	# information on available options.

	# Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
	# such as FTP and Heroku are also available. See the documentation at
	# https://docs.vagrantup.com/v2/push/atlas.html for more information.
	# config.push.define "atlas" do |push|
	#	 push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
	# end

	# Enable provisioning with a shell script. Additional provisioners such as
	# Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
	# documentation for more information about their specific syntax and use.
	#
	# NB provisioning script adapted from https://gist.github.com/alettieri/7342653
	config.vm.provision "shell", inline: <<-SHELL

		# Function for outputting progress / logging information
		function say {
			printf "\n--------------------------------------------------------\n"
			printf "\t$1"
			printf "\n--------------------------------------------------------\n"
		}

		say "Updating apt-get"
		apt-get update

		say "Installing Apache and setting it up."
		apt-get install -y apache2
		rm -rf /var/www
		ln -fs /vagrant/www /var/www
		a2enmod rewrite

		say "Installing MySQL."
		export DEBIAN_FRONTEND=noninteractive
		apt-get install -y mysql-server >/dev/null 2>&1
		sed -i -e 's/127.0.0.1/0.0.0.0/' /etc/mysql/my.cnf
		restart mysql
		mysql -u root mysql <<< "GRANT ALL ON *.* TO 'root'@'%'; FLUSH PRIVILEGES;"

		say "Installing miscellaneous useful and necessary packages"
		apt-get install -y build-essential software-properties-common vim curl wget tmux

		say "Installing PHP and Modules"
		apt-get install -y php5 php5-cli php5-common php5-imagick php5-imap php5-gd libapache2-mod-php5 php5-mysql php5-curl php5-mcrypt php5-curl

		say "Installing Composer"
		curl -sS https://getcomposer.org/installer | php
		mv composer.phar /usr/local/bin/composer

		say "Installing ruby"
		apt-get install -y ruby-full

		say "Installing sass"
		gem install sass

		say "Installing node.js"
		apt-get install -y	node

		say "Installing and setting up Mailcatcher"
		apt-get install -y libsqlite3-dev
		gem install mailcatcher
		echo "
			description \"Mailcatcher\"
				start on runlevel [2345]
				stop on runlevel [!2345]
				respawn

				exec /usr/bin/env $(which mailcatcher) --foreground --http-ip=0.0.0.0
			" > /etc/init/mailcatcher.conf
		service mailcatcher start
		echo "sendmail_path = /usr/bin/env $(which catchmail) -f test@local.dev" > /etc/php5/mods-available/mailcatcher.ini
		php5enmod mailcatcher

		SHELL


		# Copy the vhost file to default and reload apache - run every vagrant up
		config.vm.provision "shell", run:"always", inline: <<-SHELL

		# Function for outputting progress / logging information
		function say {
			printf "\n--------------------------------------------------------\n"
			printf "\t$1"
			printf "\n--------------------------------------------------------\n"
		}

		say "Writing default vhost file"
		cp /vagrant/default-vhost.conf /etc/apache2/sites-available/000-default.conf

		say "Reloading apache"
		service apache2 reload
	SHELL
end
