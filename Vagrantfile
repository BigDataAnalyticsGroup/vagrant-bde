#python -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    # Box Settings
    config.vm.box = "archlinux/archlinux"

    # Box name
    config.vm.define "bde_box"

    # Network Settings
    config.vm.hostname = "archlinux"
    config.vm.network "forwarded_port", guest: 8888, host: 8888 # Jupyter
    config.vm.network "forwarded_port", guest: 8000, host: 8000 # Django

    # Folder Settings
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.synced_folder "..", "/home/vagrant/shared", create: true # share parent dir since vagrant-bde is subrepo

    # Provider Settings
    config.vm.provider "virtualbox" do |vb|
        # Set a name for the machine displayed in VirtualBox
        vb.name = "bde"
        # Customize the amount of memory on the VM
        vb.memory = "2048"
        # Customize network settings on the VM
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    end

    # Provisions
    # Install system packages
    config.vm.provision "root-setup", type: "shell", privileged: true, run: "once", reboot: true, inline: <<-SHELL
        # Update system
        echo "Updating system packages and archlinux keyring..."
        pacman -Syy --noconfirm --quiet
        pacman -S --noconfirm --quiet --needed archlinux-keyring
        pacman -Su --noconfirm --quiet

        # Install development tools
        echo "Installing development tools..."
        pacman -S --noconfirm --quiet --needed base-devel git binutils tree neovim

        # Install Python, Virtualenv and Graphviz
        echo "Installing Python..."
        pacman -S --noconfirm --quiet --needed python python-pip graphviz

        # Install Docker
        echo "Installing docker..."
        pacman -S --noconfirm --quiet --needed docker
        usermod -aG docker vagrant # add vagrant to docker group, essentially granting root privileges to the user
        systemctl enable docker

        # Install jdk17
        echo "Installing JDK17..."
        pacman -S --noconfirm --quiet --needed jdk17-openjdk

        # Add .local/bin to PATH
        if ! $(grep -Fxq 'export PATH="$PATH:/home/vagrant/.local/bin"' /etc/profile)
        then
            echo 'export PATH="$PATH:/home/vagrant/.local/bin"' >> /etc/profile
        fi
    SHELL

    # Install pikaur
    config.vm.provision "pikaur", type: "shell", privileged: false, run: "once", inline: <<-SHELL
        # Install pikaur
        echo "Installing pikaur..."
        git clone https://aur.archlinux.org/pikaur.git
        cd pikaur
        makepkg -si --noconfirm --noprogressbar
        cd ..
        rm -rf pikaur
    SHELL

    # Install and configure postgresql
    config.vm.provision "postgresql", type: "shell", privileged: true, run: "once", inline: <<-SHELL
        # Install PostgreSQL
        echo "Installing PostgreSQL..."
        POSTGRES_DATA=/var/lib/postgres/data
        pacman -S --noconfirm --quiet --needed postgresql

        # Configuration
        if [ -d "$POSTGRES_DATA" ] && [ ! -n "$(ls -A $POSTGRES_DATA)" ];
        then
            sudo -u postgres initdb -D "$POSTGRES_DATA"
        fi
        systemctl enable postgresql --now
    SHELL

    ## Configure AGE docker image
    config.vm.provision "age", type: "shell", privileged: true, run: "once", inline: <<-SHELL
        # Install AGE
        echo "Installing AGE..."
        docker pull apache/age
        docker run --name age -p 5455:5432 -e POSTGRES_USER=postgresUser -e POSTGRES_PASSWORD=postgresPW -e POSTGRES_DB=postgresDB -d apache/age
        echo "docker start age >/dev/null" >> /home/vagrant/.bashrc
    SHELL

    # Install Python packages
    config.vm.provision "python-packages", type: "shell", privileged: false, run: "once", inline: <<-SHELL
        # Install Python packages
        echo "Installing Python packages..."
        pip install --user --break-system-packages -r shared/requirements.txt # use requirements file from shared parent dir

        jupyter contrib nbextension install --user
        jupyter nbextension enable varInspector/main
    SHELL

    # Install Neo4j docker image
    config.vm.provision "neo4j", type: "shell", privileged: true, run: "once", inline: <<-SHELL
        # Install Neo4j
        echo "Installing Neo4j..."
        docker pull neo4j:5.18-community
        docker run --name neo4j --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --env=NEO4J_AUTH=none -d neo4j:5.18-community
        echo "docker start neo4j >/dev/null" >> /home/vagrant/.bashrc
    SHELL
end
