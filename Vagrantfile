# -*- mode: ruby -*-
# vi: set ft=ruby :



Vagrant.configure("2") do |config|

    #Enable symlinks in shared folders for Virtualbox
    config.vm.provider "virtualbox" do |v|
        v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end

    #Setup
    config.vm.box = "scotch/box"
    config.vm.network "private_network", ip: "192.168.33.10"
    config.vm.hostname = "scotchbox"
    config.vm.synced_folder ".", "/var/www", :mount_options => ["dmode=777", "fmode=666"]

    config.vm.network :forwarded_port, guest: 80, host: 80

    config.vm.network :forwarded_port, guest: 33306, host: 33306

    #Create VHosts for each domain specified
    config.vm.provision "shell", inline: <<-SHELL

        ## Only thing you probably really care about is right here
        DOMAINS=("gspeak.ca" "portfolio.com")

        ## Loop through all sites
        for ((i=0; i < ${#DOMAINS[@]}; i++)); do

            ## Current Domain
            DOMAIN=${DOMAINS[$i]}

            echo "Creating directory for $DOMAIN..."
            mkdir -p /var/www/$DOMAIN/public

            echo "Creating vhost config for $DOMAIN..."
            sudo cp /etc/apache2/sites-available/scotchbox.local.conf /etc/apache2/sites-available/$DOMAIN.conf

            echo "Updating vhost config for $DOMAIN..."
            sudo sed -i s,scotchbox.local,$DOMAIN,g /etc/apache2/sites-available/$DOMAIN.conf
            sudo sed -i s,/var/www/public,/var/www/$DOMAIN/public,g /etc/apache2/sites-available/$DOMAIN.conf

            echo "Enabling $DOMAIN. Will probably tell you to restart Apache..."
            sudo a2ensite $DOMAIN.conf

            echo "So let's restart apache..."
            sudo service apache2 restart

        done

    SHELL

    #Do Chef provisioning
	config.vm.provision "chef_solo" do |chef|
		chef.add_recipe "env_var"
	end
    
    # Optional NFS. Make sure to remove other synced_folder line too
    #config.vm.synced_folder ".", "/var/www", :nfs => { :mount_options => ["dmode=777","fmode=666"] }

end
