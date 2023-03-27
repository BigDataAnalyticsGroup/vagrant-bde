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

    # Folder Settings
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.synced_folder "./shared", "/home/vagrant/shared", create: true

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

    ## Configure docker
    config.vm.provision "docker", type: "shell", privileged: true, run: "once", inline: <<-SHELL
        echo "Configuring docker..."
        docker pull apache/age
        docker run --name age -p 5455:5432 -e POSTGRES_USER=postgresUser -e POSTGRES_PASSWORD=postgresPW -e POSTGRES_DB=postgresDB -d apache/age
        echo "docker start age >/dev/null" >> /home/vagrant/.bashrc
    SHELL

    # Install Python packages
    config.vm.provision "python-packages", type: "shell", privileged: false, run: "once", inline: <<-SHELL
        # Install Python packages
        echo "Installing Python packages..."

        pip install altair==4.2.2 \
                    duckdb==0.7.1 \
                    getmac==0.9.3 \
                    graphviz==0.20.1 \
                    ipycanvas==0.13.1 \
                    ipykernel==6.22.0 \
                    ipython-genutils==0.2.0 \
                    ipython==8.12.0 \
                    ipywidgets==8.0.6 \
                    jupyter-client==7.3.4 \
                    jupyter-contrib-nbextensions==0.7.0 \
                    matplotlib==3.7.1 \
                    monotonic==1.6 \
                    notebook==6.5.3 \
                    numpy==1.24.2 \
                    pandas==1.5.3 \
                    pandasql==0.7.3 \
                    psycopg==3.1.8 \
                    py2neo==2021.2.3 \
                    scikit-learn==1.2.2 \
                    scipy==1.10.1 \
                    scrypt==0.8.20 \
                    seaborn==0.12.2 \
                    sqlparse==0.4.3 \
                    SQLAlchemy==1.4.46 \
                    tqdm==4.65.0 \
                    vega-datasets==0.9.0

        jupyter contrib nbextension install --user
        jupyter nbextension enable varInspector/main
    SHELL

    # Install and configure Neo4j
    config.vm.provision "neo4j-install", type: "shell", privileged: true, run: "once", inline: <<-SHELL
        # Install Neo4j
        echo "Installing Neo4j..."
        pikaur -S --noconfirm --noprogressbar neo4j-community

        neo4j-admin dbms set-initial-password secretpassword # needs root privileges
        systemctl enable neo4j --now
    SHELL
end
