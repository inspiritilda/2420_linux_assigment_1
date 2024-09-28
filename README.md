# Setting up a DigitalOcean droplet using `doctl` and `cloud-init`

## Table of Contents
- [Setting up a DigitalOcean droplet using `doctl` and `cloud-init`](#setting-up-a-digitalocean-droplet-using-doctl-and-cloud-init)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
    - [Prerequisites](#prerequisites)
  - [Installing and Setting up `doctl`](#installing-and-setting-up-doctl)
    - [To Install `doctl`](#to-install-doctl)
      - [Here's a breakdown of the command:](#heres-a-breakdown-of-the-command)
    - [Generating API token](#generating-api-token)
      - [To generate a personal access token](#to-generate-a-personal-access-token)
    - [Using the API token to grant account access to `doctl`](#using-the-api-token-to-grant-account-access-to-doctl)
      - [Here's a breakdown of the command:](#heres-a-breakdown-of-the-command-1)
    - [Validating `doctl` is working](#validating-doctl-is-working)
      - [Here's a breakdown of the command:](#heres-a-breakdown-of-the-command-2)
  - [Setting up a SSH key](#setting-up-a-ssh-key)
    - [Understanding SSH Keys](#understanding-ssh-keys)
    - [Generating SSH Key Pair](#generating-ssh-key-pair)
      - [Here's a breakdown of the command:](#heres-a-breakdown-of-the-command-3)
    - [Adding SSH Key to DigitalOcean](#adding-ssh-key-to-digitalocean)
      - [Here’s a breakdown of the command:](#heres-a-breakdown-of-the-command-4)
  - [Configuring `cloud-init`](#configuring-cloud-init)
    - [What is `cloud-init`?](#what-is-cloud-init)
    - [Sample `cloud-init` Configuration File](#sample-cloud-init-configuration-file)
      - [Here’s a breakdown of the command:](#heres-a-breakdown-of-the-command-5)
  - [Deploying the droplet](#deploying-the-droplet)
    - [Droplet Creation Command](#droplet-creation-command)
      - [Here’s a breakdown of the command:](#heres-a-breakdown-of-the-command-6)
  - [Verifying everything worked](#verifying-everything-worked)
  - [References](#references)

## Introduction
This guide will show you how to create an Arch Linux droplet on DigitalOcean using `doctl` command-line tool and `cloud-init` (DigitalOcean, n.d.).
We will go step by step to set up SSH keys for secure access, and automate the setup process with `cloud-init`.

### Prerequisites
- A DigitalOcean account.
- A basic understanding of how to use the Linux command line.
- This guide assumes you already have an Arch Linux droplet and can SSH into it.

## Installing and Setting up `doctl`
`doctl` is the official DigitalOcean CLI (Command Line Interface) tool, and it makes it super easy to manage everything right from your terminal (DigitalOcean, n.d.).

### To Install `doctl`
On Arch Linux, install `doctl` with the pacman package manager. You can run:
```bash
sudo pacman -S doctl
```
This installs the `doctl` package on your Arch Linux system using the pacman package manager.

#### Here's a breakdown of the command:
- `sudo`: This command runs the following command with superuser (administrator) privileges. It is necessary for installing software and making system-level changes.
- `pacman`: This is the package manager for Arch Linux. It is used to install, remove, and manage software packages on Arch-based systems.
- `-S`: This option tells `pacman` to synchronize packages, specifically to install the package specified next.
- `doctl`: This is the name of the package you want to install. In this case, it refers to the official DigitalOcean command-line tool used for managing resources on DigitalOcean.

![installing doctl](images/installing%20doctl.png)
The screenshot above shows the installation of `doctl`.

### Generating API token
Before using `doctl`, you will have to give it access to your DigitalOcean account. In order to do that, you will need to generate an API token on the DigitalOcean website. This is the only step that needs to be done outside the Arch Linux droplet.

#### To generate a personal access token
  1. Log-in to your DigitalOcean Control Panel.
  2. On the left menu, click API (this takes you to the "Applications & API" page under the Tokens tab).
  3. In the Personal access tokens section, click the Generate New Token button.
  4. Give your token a name.
  5. Choose the preferred expiration date of your token.
  6. Click Full Access in the Scopes section.
  7. Then, click Generate Token.
  8. You will receive your own personal access Token just like the screenshot below.
![personal access token](images/new%20personal%20token.png)

### Using the API token to grant account access to `doctl`
Now that you have your API token, you can use it to link `doctl` to your DigitalOcean account. Here is what you should run:
```bash
doctl auth init
```
This command initializes the `doctl` tool by linking it to your DigitalOcean account, prompting you to enter your API token for authentication.

#### Here's a breakdown of the command:
- `doctl`: This is the command-line tool for interacting with DigitalOcean's API. It allows users to manage DigitalOcean resources directly from the terminal.
- `auth`: This is a subcommand within `doctl` that deals with authentication. It manages how `doctl` connects to your DigitalOcean account.
- `init`: This command initializes the authentication process. It prompts you to enter your DigitalOcean API token, linking `doctl` to your account for subsequent operations.

Enter your access token to validate like the screenshot below.
![validating token](images/validating%20token.png)

### Validating `doctl` is working
To confirm that you have successfully linked `doctl` to your account, you can look at your account details by running:
```bash
doctl account get
```
This command retrieves and displays details about your DigitalOcean account, including the email associated with the account, droplet limits, and account status.

#### Here's a breakdown of the command:
- `doctl`: This is the command-line interface (CLI) tool for DigitalOcean, used to manage and interact with DigitalOcean resources from the terminal.
- `account`: This subcommand refers to actions related to your DigitalOcean account, providing details and information about it.
- `get`: This command retrieves information about the currently authenticated account. It displays details such as the email address associated with the account, the droplet limit, and whether the email has been verified.

If successful, the output looks like:
![validate doctl](images/validate%20account%20link.png)

## Setting up a SSH key
### Understanding SSH Keys
Secure Shell keys are great for secure, passwordless access to your server (Arch Linux, n.d.). SSH keys are a pair of cryptographic keys used for authenticating secure connections. Unlike passwords, SSH keys provide a more secure method of authentication as they are not transmitted over the network, making them resistant to brute-force attacks (Sobel, 2020). By using SSH keys, you can log in to your server without the need for a password, enhancing security.

### Generating SSH Key Pair
```bash
ssh-keygen -t ed25519 -f ~/.ssh/<key name> -C <youremail@email.com>
```
This command generates a new SSH key pair. Follow the prompts to save it in the default location.

#### Here's a breakdown of the command:
- `ssh-keygen`: This is the command used to generate a new SSH key pair.
- `-t ed25519`: The `-t` flag specifies the type of key to create. `ed25519` is the type of algorithm used for the SSH key.
- `-f ~/.ssh/<key name>`: This tells the system to sae the key files in the `~/.ssh/` directory under the given name `<key name>`. The tilde (`~`) represents the user's home directory, and `.ssh` is a hidden folder typically used to store SSH-related files.
- `-C <youremail@email.com>`: The `-C` flag is used to add a comment to the SSHkey for identification purposes. `<youremail@email.com>` is typically used as the comment, allowing you to easily identify the key.

### Adding SSH Key to DigitalOcean
Once your SSH key is generated, add it to your DigitalOcean account with:
```bash
doctl compute ssh-key import <key identifier> --public-key-file ~/.ssh/<key name>.pub
```
#### Here’s a breakdown of the command:
- `compute ssh-key create`: `compute` is a category of `doctl` commands used to manage compute resources (like droplets) on DigitalOcean. `ssh-key import` is a subcommand under `compute` that imports an SSH public key into your DigitalOcean account, allowing you to use this key for authentication on droplets.
- `<key identifier>`: This is the name or identifier you want to give to the SSH key within DigitalOcean for easy recognition. You can choose any name you like, and it will help you manage your keys, especially if you have multiple SSH keys associated with your account. This name will show up in the DigitalOcean control panel and API.
- `--public-key-file ~/.ssh/<key name>.pub`: `--public-key-file` is a flag that specifies the path to the public key file you want to import. `~/.ssh/<key name>.pub` is the file path to the SSH public key you are importing.

## Configuring `cloud-init`
### What is `cloud-init`?
`cloud-init` is a tool used in cloud environments to automate the initial setup of virtual machine (Canonical, n.d.; DigitalOcean, n.d.). It allows users to configure settings like network configuration, user creation, and package installation during the first boot of the instance (Canonical, n.d.; Cloud-init, n.d.). This automation reduces manual setup time and ensures consistency across deployments.

### Sample `cloud-init` Configuration File
After you upload your public SSH key to your DigitalOcean account, create a file named `cloud-config.yml` with the following content:
```bash
#cloud-config
users:
  - name: user-name
    primary_group: group-name
    groups: wheel
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-ed25519 ...

packages:
  - ripgrep
  - rsync
  - neovim
  - fd
  - less
  - man-db
  - bash-completion
  - tmux

disable_root: true
```

#### Here’s a breakdown of the command:
* `users`: This section specifies the user accounts to be created on the droplet.
* `- name: user-name`: Creates a user account with the specific username(`user-name`). This comment indicates that this should be replaced with the desired username.
* `primary_group: group-name`: Assigns the specified primary group to the user. This should be changed to the desired group name. The primary group determines the default group the user belongs to.
* `groups: wheel`: Adds the user to the `wheel` group, which is typically used in Unix-like systems to grant users the ability to use the `sudo` command. This allows the user to perfrom administrative tasks.
* `shell: /bin/bash`: Sets the user's default shell to `bash`, which is a common command-line interface in Linux environments.
* `sudo: ['ALL=(ALL) NOPASSWD:ALL']`: Configures the user to run any command with `sudo` without being prompted for a password. This can simplify automated scripts and commands that require administrative privileges.
* `ssh-authorized-keys:`: This section allows you to specify SSH public keys that are authorized for the user to log in without a password.
* `- ssh-ed25519 ...`: Specifies the SSH public key to be added for this user. The `ssh-ed25519` is a type of SSH key. The ellipsis (`...`) should be replaced with the actual public key.
* `packages:`: This section lists the packages that should be installed on the droplet during its initialization.
  - `- ripgrep`: A fast search tool that recursively searches your current directory for a regex pattern.
  - `- rsync`: A utility for efficiently transferring and synchronizing files across systems.
  - `- neovim`: An extensible text editor that serves as an improvement to the popular Vim editor.
  - `- fd`: A simple, fast, and user-friendly alternative to `find`, which helps locate files and directories.
  - `- less`: A terminal pager program that allows you to view (but not change) the contents of files one screen at a time.
  - `- man-db`: The manual page database for Linux, allowing you to view documentation about commands and programs using the `man` command.
  - `- bash-completion`: A package that provides command-line auto-completion for Bash, making it easier to work with commands and options.
  - `- tmux`: A terminal multiplexer that allows you to create and manage multiple terminal sessions within a single window.
* `disable_root: true`: Disables the root user account for security reasons, preventing direct root login and encouraing the use of the `sudo` command for adminitrative tasks instead.

## Deploying the droplet
Now that everything is configured, you're ready to deploy your Arch Linux droplet using `doctl` and the `cloud-init` configuration file.

### Droplet Creation Command
To create the droplets, run the following command:
``` bash
doctl compute droplet create --image arch-linux-2024-01-01 --size s-1vcpu-1gb --region nyc1 --ssh-keys git-user --user-data-file <path-to-your-cloud-init-file> --wait first-droplet second-droplet
```
#### Here’s a breakdown of the command:
* `doctl compute droplet create`: This is the base command used to create a new droplet.
* `--image arch-linux-2024-01-01`: This option specifies the operating system image for the droplets, which in this case is arch-linux-2024-01-01.
* `--size s-1vcpu-1gb`: This specifies the resources for each droplet. Here, each droplet is allocated one virtual CPU and 1 GB of RAM.
* `--region nyc1`: This option defines the region for deploying the droplets. In this example, the droplets will be created in the NYC1 datacenter.
* `--ssh-keys`: This parameter allows you to import SSH keys into the droplet from your DigitalOcean account. You can list available keys using the command `doctl compute ssh-key list`.
* `--user-data-file <path-to-your-cloud-init-file`: This specifies the path to your `cloud-config.yaml file`. For instance, it could look something like `/Users/example-user/cloud-config.yaml`.
* `--wait`: This ensures that the command will not return until the droplet is fully created and ready for use.
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

To verify that the `nginx` configuration is functioning, simply copy one of the droplet's public IP addresses and paste it into your web browser, then press Enter. You should see the nginx homepage, which will display the name of your droplet along with its IP address. This indicates that your setup was successful.

## References
1. Arch Linux. (n.d.). *Installation guide*. Retrieved from [https://wiki.archlinux.org/title/Installation_guide](https://wiki.archlinux.org/title/Installation_guide)  
   In-text citation: (Arch Linux, n.d.)

2. Arch Linux. (n.d.). *SSH keys*. Retrieved from [https://wiki.archlinux.org/title/Secure_Shell#SSH_keys](https://wiki.archlinux.org/title/Secure_Shell#SSH_keys)  
   In-text citation: (Arch Linux, n.d.)

3. Canonical. (n.d.). *Cloud-init*. Retrieved from [https://cloud-init.readthedocs.io/en/latest/](https://cloud-init.readthedocs.io/en/latest/)  
   In-text citation: (Canonical, n.d.)

4. Chacon, S., & Straub, B. (2017). *Pro Git* (2nd ed.). Apress. Retrieved from [https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)  
   In-text citation: (Chacon & Straub, 2017)

5. DigitalOcean. (n.d.). *How to create and manage SSH keys on DigitalOcean*. Retrieved from [https://www.digitalocean.com/docs/ssh/create-ssh-keys/](https://www.digitalocean.com/docs/ssh/create-ssh-keys/)  
   In-text citation: (DigitalOcean, n.d.-a)

6. DigitalOcean. (n.d.). *How to use cloud-init to configure a droplet*. Retrieved from [https://www.digitalocean.com/docs/droplets/how-to/use-cloud-init/](https://www.digitalocean.com/docs/droplets/how-to/use-cloud-init/)  
   In-text citation: (DigitalOcean, n.d.-b)

7. DigitalOcean. (n.d.). *How to Perform Initial Server Configuration with Cloud-Init*. Retrieved from [https://www.digitalocean.com/community/tutorials/how-to-use-cloud-config-for-your-initial-server-setup](https://www.digitalocean.com/community/tutorials/how-to-use-cloud-config-for-your-initial-server-setup)  
   In-text citation: (DigitalOcean, n.d.-c)

8. GNU. (n.d.). *Bash completion*. Retrieved from [https://bash-completion.alioth.debian.org/](https://bash-completion.alioth.debian.org/)  
   In-text citation: (GNU, n.d.)

9. OpenSSH. (n.d.). *SSH key management*. Retrieved from [https://www.openssh.com/manual.html](https://www.openssh.com/manual.html)  
   In-text citation: (OpenSSH, n.d.)

10. Sobel, M. (2020). *Learning Linux for iOS development: An introduction to Unix-based development for mobile applications*. Packt Publishing.  
    In-text citation: (Sobel, 2020)