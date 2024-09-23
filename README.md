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
  - [Setting up a SSH key](#setting-up-a-ssh-key)
    - [Generate SSH Keys:](#generate-ssh-keys)
    - [Add SSH Key to DigitalOcean:](#add-ssh-key-to-digitalocean)
  - [Configuring cloud-init](#configuring-cloud-init)
    - [Sample cloud-init Configuration File:](#sample-cloud-init-configuration-file)
  - [Deploying the droplet](#deploying-the-droplet)
    - [Droplet Creation Command:](#droplet-creation-command)
  - [Verifying everything worked](#verifying-everything-worked)
    - [Check Droplet Status:](#check-droplet-status)
    - [Connect to Your Droplet via SSH:](#connect-to-your-droplet-via-ssh)
  - [Uploading a custom image to DigitalOcean](#uploading-a-custom-image-to-digitalocean)
    - [Steps to Upload a Custom Image:](#steps-to-upload-a-custom-image)

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

## Setting up a SSH key
SSH keys are great for secure, passwordless access to your server.
### Generate SSH Keys:
To create a new SSH key pair on your local machine, run:
```bash
doctl compute ssh-key create <key-name>
```
* Make sure give your SSH a name by replacing `<key-name>`.
### Add SSH Key to DigitalOcean:
Next, to add your SSH key to DigitalOcean, use this command:
```bash
doctl compute ss-key
```

## Configuring cloud-init
`cloud-init` is super handy for automating the setup of your droplet right when it is created. We will use it to create a new user, install some packages, and set up SSH access without having to do it all manually.
### Sample cloud-init Configuration File:
After you upload your public SSH key to your DigitalOcean account, create a file named `cloud-config.yml` with the following content:
```bash
#cloud-config
users:
  - name: example-user
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_import_id:
      - gh:<your-GitHub-username>
disable_root: true
packages:
  - nginx
runcmd:
  - 'export PUBLIC_IPV4=$(curl -s http://169.254.169.254/metadata/v1/interfaces/public/0/ipv4/address)'
  - 'echo Droplet: $(hostname), IP Address: $PUBLIC_IPV4 > /var/www/html/index.html'
```
The YAML file for this tutorial defines:
* A new user (`example-user`) account on the Droplet with root-level permissions and the user’s preferred shell (`bash`). It also specifies the SSH key to import from GitHub and associates with the new user account.
* An option to disable root user access so that only the `example-user` user can access the Droplet.
* Which packages to install upon deployment, in this case `nginx`.
* Two commands that create an environment variable and configure the nginx server.

## Deploying the droplet
Now that everything is set up, you can deploy your Arch Linux droplet using doctl and the cloud-init configuration file.
### Droplet Creation Command:
``` bash
doctl compute droplet create --image ubuntu-22-04-x64 --size s-1vcpu-1gb --region nyc1 --ssh-keys git-user --user-data-file <path-to-your-cloud-init-file> --wait first-droplet second-droplet
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