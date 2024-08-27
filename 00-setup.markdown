---
layout: page
title: Setup
permalink: /setup/
---

<nav>
  <h4>**Contents**</h4>
  * toc goes here
  {:toc}
</nav>

# Requirements

## 1. BASH

You should have a basic working knowledge of Bash shell. If you need a refresher, please check out:

[https://swcarpentry.github.io/shell-novice/index.html](https://swcarpentry.github.io/shell-novice/index.html)

## 2. Laptop and SSH

You will need to bring a laptop on the day of the workshop to work on exercises. 

A VM on the [Nectar Research Cloud](https://ardc.edu.au/services/ardc-nectar-research-cloud/) will be allocated for you to use. On the day of the workshop, you will receive the IP address, username and password for the VM.

To access the VM, you will need an SSH (Secure Shell) client on your machine. 

### **Windows**

The latest builds of Windows 10 and 11 come with SSH server and client. To see if your windows installation has SSH:
- start Command Prompt (cmd) or Powershell
- type in 'ssh'

If SSH is available, you should see something like 

```
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface]
           [-b bind_address] [-c cipher_spec] [-D [bind_address:]port]
           [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11]
           [-i identity_file] [-J [user@]host[:port]] [-L address]
           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]
           [-Q query_option] [-R address] [-S ctl_path] [-W host:port]
           [-w local_tun[:remote_tun]] destination [command]
```

If SSH is not available, please install [PuTTY](https://www.putty.org/).

Open PuTTY and connect using the IP address of your VM as the host name, and set port to 22.

When prompted, type in your username and password.


### **macOS**

Use `ssh` in the *Terminal* app. For example, if your IP is `0.0.0.0` and user name is `user`, you would enter:

```
ssh user@0.0.0.0
```

When prompted, type your password and then press `return`.


### **Linux** 

Same as macOS, an SSH client should be included. If not, install via the default package manager. e.g.:

```
# ubuntu
sudo apt install openssh-client
```

## 3. VSCode (optional)

You will need to do some text editing during the workshop. 
If you are comfortable with Linux CLI text editors (e.g. vim, nano), feel free to ignore this part. 

If you would like to use a GUI text editor, we recommend using installing [Visual Studio Code](https://code.visualstudio.com/), and also the official SSH extension (Remote-SSH).

In short:

1. Download and install VS Code (available for Windows, Mac, and various Linux OSes).
2. Start VS Code, install Remote-SSH extension. 
   * Press F1 or Ctrl+Shift+P, search for "Extensions: install extensions"
   * search for "Remote-SSH", click on "install"

After extension is installed, to connect to a remote server:
1. Press F1 or Ctrl+Shift+P, search for "Remote-SSH: Connect to Host..."
2. Enter "[username]@[host.ip.address]

See these articles for more details:
- [https://code.visualstudio.com/docs/remote/ssh-tutorial]()
- [https://code.visualstudio.com/docs/remote/ssh]()

If using VSCode, You may also find the [Nextflow language plugin](https://marketplace.visualstudio.com/items?itemName=nextflow.nextflow) helpful.

## 4. Slack channel

We will be using Slack for some communication before and during the workshop. 

If you have an account on the Adelaide Bioinformatics Hub slack workspace ([bioinformaticshubsa.slack.com]()), please go ahead and join the [#nextflow-snakemake-24](bioinformaticshubsa.slack.com#nextflow-snakemake-24) channel.

If you are not yet part of the workspace, please contact one of the event organisers.
