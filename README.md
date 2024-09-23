# Setting up a DigitalOcean droplet using doctl and cloud-init

## Table of Contents
- [Setting up a DigitalOcean droplet using doctl and cloud-init](#setting-up-a-digitalocean-droplet-using-doctl-and-cloud-init)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
    - [Prerequisites:](#prerequisites)
  - [Installing and Setting up doctl](#installing-and-setting-up-doctl)
  - [Authenticate `doctl`:](#authenticate-doctl)
  - [Generating API token](#generating-api-token)
  - [Use the API token to grant account access to doctl](#use-the-api-token-to-grant-account-access-to-doctl)
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
This guide will walk you through creating an Arch Linux droplet on DigitalOcean using the command-line tool doctl and cloud-init. We will set up SSH keys for secure access, upload a custom Arch Linux image, and automate the droplet setup using cloud-init.

### Prerequisites:
- A basic understanding of Linux command-line usage.
- A DigitalOcean account.
- Your local machine should have ssh installed.
- Familiarity with using git for version control.

##  Installing and Setting up doctl
`doctl` is the official DigitalOcean CLI tool that allows you to manage your resources from the command line.

Steps to Install doctl:
If you're using Arch Linux, you can install doctl using the pacman package manager:

```bash
sudo pacman -S doctl
```

## Authenticate `doctl`:
Once installed, you need to authenticate doctl with your DigitalOcean account:

```bash
doctl auth init
```

## Generating API token
You will be prompted to enter your API token, which can be generated from the DigitalOcean control panel under API > Generate New Token.

To generate a personal access token, log in to the DigitalOcean Control Panel:

1. In the left menu, click API, which takes you to the Applications & API page on the Tokens tab.
2. In the Personal access tokens section, click the Generate New Token button.

## Use the API token to grant account access to doctl
Use the API token to grant `doctl` access to your DigitalOcean account. Pass in the token string when prompted by `doctl auth init`, and give this authentication context a name.
```bash
doctl auth init --context <NAME>
```

Authentication contexts let you switch between multiple authenticated accounts. You can repeat steps 2 and 3 to add other DigitalOcean accounts, then list and switch between authentication contexts:
```bash
doctl auth list
doctl auth switch --context <NAME>
```

## Validate that doctl is working
To confirm that you have successfully authorized `doctl`, review your account details by running:
```bash
doctl account get
```

If successful, the output looks like:
```bash
Email                      Droplet Limit    Email Verified    UUID                                        Status
sammy@example.org          10               true              3a56c5e109736b50e823eaebca85708ca0e5087c    active
```

To confirm that you have successfully granted write access to `doctl`, create an Ubuntu 23.10 Droplet in the SFO2 region by running:
```bash
doctl compute droplet create --region sfo2 --image ubuntu-23-10-x64 --size s-1vcpu-1gb <DROPLET-NAME>
```

You can get a full list of available Droplet images by running doctl compute image list. The output of that command includes an ID column with the new Droplet’s ID. For example:
```bash
ID           Name            Public IPv4    Private IPv4    Public IPv6    Memory    VCPUs    Disk    Region    Image                       Status    Tags    Features    Volumes
187949338    droplet-name                                                  1024      1        25      sfo2      Ubuntu 18.04.3 (LTS) x64    new
```

Use that value to delete the Droplet by running:
```bash
doctl compute droplet delete <DROPLET-ID>
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