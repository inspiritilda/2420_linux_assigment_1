# Setting up a DigitalOcean droplet using doctl and cloud-init

## Table of Contents
- [Setting up a DigitalOcean droplet using doctl and cloud-init](#setting-up-a-digitalocean-droplet-using-doctl-and-cloud-init)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
    - [Prerequisites:](#prerequisites)
  - [Installing and Setting up doctl](#installing-and-setting-up-doctl)
    - [Steps to Install doctl:](#steps-to-install-doctl)
  - [Generating API token](#generating-api-token)
    - [To generate a personal access token:](#to-generate-a-personal-access-token)
    - [Use the API token to grant account access to doctl](#use-the-api-token-to-grant-account-access-to-doctl)
  - [Authenticate `doctl`:](#authenticate-doctl)
  - [Validate that doctl is working](#validate-that-doctl-is-working)
  - [Configuring cloud-init](#configuring-cloud-init)
    - [Sample cloud-init Configuration File:](#sample-cloud-init-configuration-file)
  - [Uploading a custom image to DigitalOcean](#uploading-a-custom-image-to-digitalocean)
    - [Steps to Upload a Custom Image:](#steps-to-upload-a-custom-image)
  - [Setting up a SSH key](#setting-up-a-ssh-key)
    - [Generate SSH Keys:](#generate-ssh-keys)
    - [Add SSH Key to DigitalOcean:](#add-ssh-key-to-digitalocean)
  - [Deploying the droplet](#deploying-the-droplet)
    - [Droplet Creation Command:](#droplet-creation-command)
  - [Verifying everything worked](#verifying-everything-worked)
    - [Check Droplet Status:](#check-droplet-status)
    - [Connect to Your Droplet via SSH:](#connect-to-your-droplet-via-ssh)

## Introduction
This guide will show you how to create an Arch Linux droplet on DigitalOcean using `doctl` command-line tool and `cloud-init`.
We will go step by step to set up SSH keys for secure access, upload a custom Arch Linux image, and automate the setup process with `cloud-init`.

### Prerequisites:
- A DigitalOcean account.
- `ssh` installed on your local machine.
- A basic understanding of how to use the Linux command line.

##  Installing and Setting up doctl
`doctl` is the official DigitalOcean CLI tool, and it makes it super easy to manage everything right from your terminal.

### Steps to Install doctl:
On Arch Linux, install `doctl` with the pacman package manager. You can run:
```bash
sudo pacman -S doctl
```

## Generating API token
When you run the authentication, it will ask for an API token.

### To generate a personal access token:
1. Log-in to your DigitalOcean Control Panel.
2. On the left menu, click API (this takes you to the "Applications & API" page under the Tokens tab).
3. In the Personal access tokens section, click the Generate New Token button.
4. You will receive your own personal access Token jsut like the screenshot below.

![personal access token](images/new%20personal%20token.png)

### Use the API token to grant account access to doctl
Now that you have your API token, you can use it to link `doctl` to your DigitalOcean account. When you run `doctl auth init`, just enter the token when it asks for it and give this authentication context a name if you would like. It looks like this:
```bash
doctl auth init --context <NAME>
```
![validating token](images/validating%20token.png)

## Authenticate `doctl`:
Once you have installed `doctl`, you will need to link it to your DigitalOcean account. To do that, run:
```bash
doctl auth init
```
![authenticate doctl](images/authenticate%20doctl.png)

## Validate that doctl is working
To confirm that you have successfully linked `doctl` to your account, you can look at your account details by running:
```bash
doctl account get
```
If successful, the output looks like:
```bash
Email                      Droplet Limit    Email Verified    UUID                                        Status
sammy@example.org          10               true              3a56c5e109736b50e823eaebca85708ca0e5087c    active
```

## Configuring cloud-init
`cloud-init` allows you to automate the initial setup of your droplet. We’ll use it to create a user, install some packages, and configure SSH access.

### Sample cloud-init Configuration File:
Create a file named `cloud-config.yml` with the following content:
```bash
#cloud-config
users:
  - name: newuser
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    ssh-authorized-keys:
      - ssh-rsa AAAAB3...your-public-ssh-key
packages:
  - vim
  - htop
  - git
ssh_pwauth: false
disable_root: true
```

This configuration does the following:

* Creates a new user `newuser` with sudo privileges.
* Installs basic packages such as `vim`, `htop`, and `git`.
* Adds your SSH key for passwordless login.
* Disables root access via SSH for added security.

Save this file as `cloud-config.yml`.

## Uploading a custom image to DigitalOcean
To create a droplet running Arch Linux, you first need to upload a custom Arch Linux image to your DigitalOcean account.

### Steps to Upload a Custom Image:
1. run `doctl compute image create`. Basic usage looks like this, but you can read the usage docs for more details:
```bash
doctl compute image create <image-name> [flags]
```
2. The following example creates a custom image named "Example Image" from a URL and stores it in the `nyc1` region:
```bash
doctl compute image create "Example Image" --image-url "https://example.com/image.iso" --region nyc1
```

## Setting up a SSH key
SSH keys provide secure, passwordless authentication to your server.

### Generate SSH Keys:
To create a new SSH key pair on your local machine, run:
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

* `The -t rsa` flag specifies the RSA algorithm.
* `The -b 4096` flag creates a 4096-bit key for enhanced security.
* 
You can find your public key at `~/.ssh/id_rsa.pub`. Copy this key so it can be added to your droplet.

### Add SSH Key to DigitalOcean:
Go to Settings > Security in your DigitalOcean account.
Click Add SSH Key and paste the contents of your `id_rsa.pub` file.

## Deploying the droplet
Now that everything is set up, you can deploy your Arch Linux droplet using doctl and the cloud-init configuration file.

### Droplet Creation Command:
``` bash
doctl compute droplet create "my-arch-droplet" \
--region nyc3 \
--size s-1vcpu-1gb \
--image <your-image-id> \
--ssh-keys <your-ssh-key-fingerprint> \
--user-data-file cloud-config.yml \
--wait
```

* `--region`: Specify the region, e.g., nyc3.
* `--size`: Define the droplet size, e.g., s-1vcpu-1gb.
* `--image`: Use the ID of your custom Arch Linux image.
* `--ssh-keys`: Add your SSH key fingerprint (which you can retrieve from the DigitalOcean control panel).
* `--user-data-file`: Pass the cloud-init file to automate the initial setup.

## Verifying everything worked
### Check Droplet Status:
Run the following command to ensure the droplet was created successfully:
```bash
doctl compute droplet list
```

### Connect to Your Droplet via SSH:
To connect to your droplet, use the IP address listed in the previous command:
```bash
ssh newuser@<your-droplet-ip>
```

If the `cloud-init` script worked correctly, you should be logged in as `newuser`, and the specified packages should be installed.