# Setting up a DigitalOcean droplet using doctl and cloud-init

# Table of Contents

# Introduction
This guide will walk you through creating an Arch Linux droplet on DigitalOcean using the command-line tool doctl and cloud-init. We will set up SSH keys for secure access, upload a custom Arch Linux image, and automate the droplet setup using cloud-init.

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



# Uploading a custom image to DigitalOcean

# Configuring cloud-init

# Setting up a SSH key

# Deploying the droplet

# Verifying everything worked