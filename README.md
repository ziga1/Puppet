# Puppet set-up for Ubuntu + Manifest creation and apply for UFW Firewall Configuration + Possible problems and solutions that might challenge you, like they did me 

## Introduction

This repository contains instructions on how to set-up Puppet to be able to create a Puppet manifest and configure the Uncomplicated Firewall (UFW) on an Ubuntu machine. The manifest enables UFW, sets the default incoming policy to deny, and allows outgoing connections to specific destinations and ports. After chapter ## Applying the manifest I present the challenges I encountered and their solutions.

## Conditions 
- Working Ubuntu machine is ready for use with sudo privileges.
- UFW is installed (`sudo apt-get install ufw`).

## Puppet set-up
- The set-up consisted of using Puppet docs located on this url - https://www.puppet.com/docs/puppet/7/install_puppet.html - and the following guide - https://phoenixnap.com/kb/install-puppet-ubuntu - *USING ONLY THE GUIDE URL -FOLLOW INSTRUCTIONS FOR SETTING-UP PUPPETSERVER SINCE THE COMMAND PUPPET APPLY APPLIES THE MANIFEST ON THE DEFAULT MACHINE/MASTER*

## Making the manifest file 
- On your Ubuntu machine use command "cd /etc/puppetlabs/code/environments/production/manifests" to enter the manifests directory and in that directory, make a new file using text editor nano with the .pp extension. You can use this command "sudo nano YOUR_FILE_NAME.pp"
- In the manifest file paste in the following code starting on line 18 and ending on line 40:

exec { 'enable-ufw':
  command     => '/usr/sbin/ufw --force enable',
  logoutput   => true,
  path        => ['/usr/sbin'],
}

exec { 'deny-all-incoming':
  command     => '/usr/sbin/ufw default deny incoming',
  logoutput   => true,
  path        => ['/usr/sbin'],
}

exec { 'allow-outgoing-srv1-443':
  command     => '/usr/sbin/ufw allow out to srv1.example.com port 443 proto tcp',
  logoutput   => true,
  path        => ['/usr/sbin'],
}

exec { 'allow-outgoing-srv1-8140':
  command     => '/usr/sbin/ufw allow out to srv1.example.com port 8140 proto tcp',
  logoutput   => true,
  path        => ['/usr/sbin'],
}

- After successfully pasting the code press ctrl+x to exit and y to save the file.
- You can edit this manifest to suit your needs, url used in this example is an "example url". To yield even better results instead of using the url in your puppet code, use the destination IP of the url you want to edit rules for.

## Applying the manifest
- In order to successfully apply the manifest we need to put in a command that consists of 2 parts.
  The first part is is responsible for running Puppet in "apply" mode, it looks like this "sudo /opt/puppetlabs/bin/puppet apply"
  The second part defines the path to the Puppet manifest file, it looks like this "/etc/puppetlabs/code/environments/production/manifests/YOUR_FILE_NAME.pp"

## Challenges with solutions
1. To make an Ubuntu machine I used Oracles VM VirtualBox because I already used up all the free features of Azure and AWS + its super user-friendly. The only problem I had with it is that I couldnt use my mouse and copy&paste feature. The solution was to set-up PuTTY and connect with my VM using ssh service.
2. When setting-up puppetserver I failed to start/activate it, the status of puppetserver was "activating" but it did not actually activate. I found out the error occurs because there is not enough RAM. To fix this problem I needed to navigate to puppetservers config file and change the line responsible for the amount of allocated RAM for the puppetserver. After that I restarted puppetserver and it worked.
3. To make puppetserver work as it is supposed to I also needed to change the contents of hosts file. To it I needed to add the following line with the IP of my machine "192.168.8.156 master-virtualbox". I found this out after trying to apply my first simple manifest, just to check if it works.
4. In the making of the manifest I used to change ufw rules I encounered a problem, where I applied the manifest but the response said it didnt recognise the ufw module. I later found out I needed to install the model I wanted to use from PuppetForge, it was not enough to simply copy, paste the model and then edit and apply it. The solution was to use the resource "exec" since I wanted to make a manifest that wouldnt need anything more but the basic puppet set-up to work.
5. The last but probably least mentionable problem I had was again, with applying the manifest. After I applied it, the response said "Notice: Applied catalog in 0.63 seconds" but the ufw status was still inactive and the rules did not change. I later found out I made a typo in the puppet code whilst writing the path to the executable binary of the ufw command. In my mind the path to ufws executable binary was bin and not sbin.
