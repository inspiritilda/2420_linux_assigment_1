# Setting up a DigitalOcean droplet using `doctl` and `cloud-init`

# Table of Contents
- [Setting up a DigitalOcean droplet using `doctl` and `cloud-init`](#setting-up-a-digitalocean-droplet-using-doctl-and-cloud-init)
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
  - [Uploading a custom image to DigitalOcean](#uploading-a-custom-image-to-digitalocean)
    - [Steps to Upload a Custom Image:](#steps-to-upload-a-custom-image)
  - [Configuring cloud-init](#configuring-cloud-init)
    - [Sample cloud-init Configuration File:](#sample-cloud-init-configuration-file)
  - [Deploying the droplet](#deploying-the-droplet)
    - [Droplet Creation Command:](#droplet-creation-command)
  - [Verifying everything worked](#verifying-everything-worked)


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
Now that everything is configured, you're ready to deploy your Arch Linux droplet using `doctl` and the `cloud-init` configuration file.
### Droplet Creation Command:
To create the droplets, run the following command:
``` bash
doctl compute droplet create --image ubuntu-22-04-x64 --size s-1vcpu-1gb --region nyc1 --ssh-keys git-user --user-data-file <path-to-your-cloud-init-file> --wait first-droplet second-droplet
```
Here is a breakdown of the command:
* `--image ubuntu-22-04-x64`: This option specifies the operating system image for the droplets, which in this case is Ubuntu 22.04.
* `--size s-1vcpu-1gb`: This specifies the resources for each droplet. Here, each droplet is allocated one virtual CPU and 1 GB of RAM.
* `--region nyc1`: This option defines the region for deploying the droplets. In this example, the droplets will be created in the NYC1 datacenter.
* `--ssh-keys`: This parameter allows you to import SSH keys into the droplet from your DigitalOcean account. You can list available keys using the command `doctl compute ssh-key list`.
* `--user-data-file <path-to-your-cloud-init-file`: This specifies the path to your `cloud-config.yaml file`. For instance, it could look something like `/Users/example-user/cloud-config.yaml`.
* `--wait`: This tells `doctl` to wait until the droplets are fully deployed before accepting any new commands.
* `first-droplet second-droplet`: These are the names of the droplets being created. You can name as many droplets as you want by adding more names at the end of the command.

After executing the command, the terminal will not display a prompt until the droplets finish deploying, which may take a few minutes. Once the deployment is successful, the output will resemble the following:
```bash
ID           Name              Public IPv4       Private IPv4    Public IPv6    Memory    VCPUs    Disk    Region    Image               VPC UUID                                Status    Tags    Features                            Volumes
311143987    second-droplet    203.0.113.199    203.0.113.4                     1024      1        25      nyc1      Ubuntu 22.04 x64    cfcbcc95-365a-4705-a18d-54abde1fc7b4    active            droplet_agent,private_networking
311912986    first-droplet     203.0.113.146    203.0.113.3                     1024      1        25      nyc1      Ubuntu 22.04 x64    cfcbcc95-365a-4705-a18d-54abde1fc7b4    active            droplet_agent,private_networking
```

## Verifying everything worked
After successfully deploying the droplets, it is important to check if the cloud-init configuration was executed properly. You can log in to one of the droplets using the username defined in the `cloud-config.yaml file`. To do this, you can use the OpenSSH command shown below. Make sure to replace the placeholder with the public IPv4 address of your droplet:
```bash
ssh example-user@<your-droplet-ip-address>
```
If everything is set up correctly, your terminal prompt should change to something like this:
```bash
example-user@first-droplet:~$
```
At this point, you are logged in and can explore the droplet.

To verify that the nginx configuration is functioning, simply copy one of the droplet's public IP addresses and paste it into your web browser, then press Enter. You should see the nginx homepage, which will display the name of your droplet along with its IP address. This indicates that your setup was successful.