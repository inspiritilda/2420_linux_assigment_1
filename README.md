# Setting up a DigitalOcean droplet using doctl and cloud-init

# Table of Contents

# Introduction
This guide will walk you through creating an Arch Linux droplet on DigitalOcean using the command-line tool doctl and cloud-init. We will set up SSH keys for secure access, upload a custom Arch Linux image, and automate the droplet setup using cloud-init.

# Uploading a custom image to DigitalOcean

# Setting up a SSH key

# Deploying the droplet

# Installing and Setting up doctl
doctl is the official DigitalOcean CLI tool that allows you to manage your resources from the command line.

Steps to Install doctl:
If you're using Arch Linux, you can install doctl using the pacman package manager:
sudo pacman -S doctl

Authenticate doctl:
Once installed, you need to authenticate doctl with your DigitalOcean account:
doctl auth init

# Generating API token
You will be prompted to enter your API token, which can be generated from the DigitalOcean control panel under API > Generate New Token.

To generate a personal access token, log in to the DigitalOcean Control Panel.

In the left menu, click API, which takes you to the Applications & API page on the Tokens tab. In the Personal access tokens section, click the Generate New Token button.

# Use the API token to grant account access to doctl
Use the API token to grant doctl access to your DigitalOcean account. Pass in the token string when prompted by doctl auth init, and give this authentication context a name.
doctl auth init --context <NAME>

Authentication contexts let you switch between multiple authenticated accounts. You can repeat steps 2 and 3 to add other DigitalOcean accounts, then list and switch between authentication contexts:
doctl auth list
doctl auth switch --context <NAME>

# Validate that doctl is working
To confirm that you have successfully authorized doctl, review your account details by running:
doctl account get

If successful, the output looks like:
Email                      Droplet Limit    Email Verified    UUID                                        Status
sammy@example.org          10               true              3a56c5e109736b50e823eaebca85708ca0e5087c    active

To confirm that you have successfully granted write access to doctl, create an Ubuntu 23.10 Droplet in the SFO2 region by running:
doctl compute droplet create --region sfo2 --image ubuntu-23-10-x64 --size s-1vcpu-1gb <DROPLET-NAME>

You can get a full list of available Droplet images by running doctl compute image list. The output of that command includes an ID column with the new Droplet’s ID. For example:
ID           Name            Public IPv4    Private IPv4    Public IPv6    Memory    VCPUs    Disk    Region    Image                       Status    Tags    Features    Volumes
187949338    droplet-name                                                  1024      1        25      sfo2      Ubuntu 18.04.3 (LTS) x64    new

Use that value to delete the Droplet by running:
doctl compute droplet delete <DROPLET-ID>

# Configuring cloud-init

# Verifying everything worked