# Setup and installation

First of all, make a clone or [fork of this repository](http://help.github.com/fork-a-repo/) and replace all occurrences of `kinotel` with a name of your choice.

## Launch an EC2 instance

[Start a "micro" Amazon EC2 instance](https://console.aws.amazon.com/ec2/home) and use one of the following AMIs, depending on where you chose to launch it:

Use this site (see dropdown on right side) to choose an updated image: http://alestic.com/

The following images are pretty old:

- US west: `ami-ad7e2ee8`
- US east: `ami-ccf405a5`
- EU west: `ami-fb9ca98f`
- Asia Pacific (Singapore): `ami-0c423c5e`

> These AMIs where taken from <http://uec-images.ubuntu.com/releases/10.10/release/>

Go with the defaults in the "wizard" presented. Chose to create a new key pair when asked and **be sure to make a secure backup of the private key** that you will download. A good place to put your private key is in `~/.ssh/kinotel.pem` and then `chmod 0600 ~/.ssh/kinotel.pem` so no one else can read it but you.

When the instance is green and "started", log in to the machine:

    ssh -i ~/.ssh/kinotel.pem ubuntu@XXX.amazonaws.com

*Note: Replace `XXX.amazonaws.com` with the hostname or address of your instance.*

*Note: If you are running Microsoft Windows, which lacks an SSH client, see [WINDOWS-SSH.md](WINDOWS-SSH.md#readme)*.

## Install software

    sudo apt-get update
    sudo apt-get install nginx git-core daemon
    sudo chown -R www-data:www-data /var/www

Node.js and NPM:

* from https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager *
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs npm

# The following aws-* packages are only necessary if you're running services on aws.

## Installed libxml2-dev in order to make aws-sdk (for node dev on aws) install

apt-get install libxml2-dev

## Instal aws-sdk

npm install aws-sdk

## Checkout your source

    sudo mkdir /var/syncpoint
    sudo chown www-data:www-data /var/syncpoint

If your git repository is public (i.e. viewable by anyone):

    //sudo -Hu www-data git clone https://github.com/you/kinotel.git /var/kinotel
      sudo -Hu www-data git clone https://github.com/chrisekelley/Syncpoint-API.git /var/syncpoint

If your git repository is private:

    sudo -Hu www-data ssh-keygen -t rsa  # chose "no passphrase"
    sudo cat /var/www/.ssh/id_rsa.pub
    # Add the key as a "deploy key" at https://github.com/you/kinotel/admin
    sudo -Hu www-data git clone git@github.com:you/kinotel.git /var/kinotel

### Following advice from 
### http://www.bennadel.com/blog/2321-How-I-Got-Node-js-Running-On-A-Linux-Micro-Instance-Using-Amazon-EC2.htm
### to install the forever module

sudo npm install forever

### configure the forever command to point to your script:

forever start -al forever.log -ao out.log -ae err.log /var/syncpoint/bin/run

# I did not need to follow the rest of these instructions

## Configure & start your services

Your Node.js web server:

    sudo ln -s /var/kinotel/init.d/kinotel-httpd /etc/init.d/
    sudo update-rc.d kinotel-httpd defaults
    sudo invoke-rc.d kinotel-httpd start
    
Optional `kinotel-processor`:
    
    sudo ln -s /var/kinotel/init.d/kinotel-processor /etc/init.d/
    sudo update-rc.d kinotel-processor defaults
    sudo invoke-rc.d kinotel-processor start


## Configure Nginx

There are three different kinds of setups to chose from:

1. `kinotel-http` -- HTTP only
2. `kinotel-https` -- HTTPS with HTTP redirecting to HTTPS
3. `kinotel-https-http` -- HTTPS and HTTP

If you are using HTTPS, make sure you have added your SSL certificate and key at `/var/kinotel/ssl/ssl.crt` and `/var/kinotel/ssl/ssl.key`.

Replace `kinotel-https` below with the configuration of your choice:

    sudo ln -sf /var/kinotel/etc/nginx/sites-available/kinotel-https \
                /etc/nginx/sites-enabled/default
    sudo invoke-rc.d nginx restart


## Done

Your web app should now be operational.

Note that the programs `kinotel-httpd` and `kinotel-processor` are written in the Move programming language (like JavaScript but simpler). [Learn more at movelang.org](http://movelang.org/).

If everything works, **continue by reading [WORKFLOW.md](https://github.com/rsms/ec2-webapp/blob/master/WORKFLOW.md#readme)**
