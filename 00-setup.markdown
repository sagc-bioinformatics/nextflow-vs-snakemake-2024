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

# Setting up your environment

Every attendee has been allocated a virtual machine on the [Nectar Research Cloud](https://ardc.edu.au/services/ardc-nectar-research-cloud/), which will be available for the duration of the workshop.

You should have received an email with the IP address of your VM, as well as your username and password.

## How do I access my VM?

If you're comfortable with using `ssh` to access a remote machine already, you can skip this step!
However, if you're unsure of how to proceed, please follow the below instructions for the platform you're connecting from:

### Windows

Please install [PuTTY](https://www.putty.org/) (an SSH client).

Open PuTTY and connect using the IP address of your VM as the host name, and set port to 22.

When prompted, type in your username and password.

### Mac

Use `ssh` in the *Terminal* app. For example, if your IP is `0.0.0.0` and user name is `user`, you would enter:

```
ssh user@0.0.0.0
```

When prompted, type your password and then press `return`.


### Linux

Open your favoured terminal emulator and use `ssh`.

# Accessing workshop data

- Instructions for where to find data / directory structure go here


# How would I set this up locally?

Although accessing these VMs via `ssh` is similar to how you might use an HPC environment,
you may wish to have Nextflow installed locally to replicate this in your own time.

Please follow the [installation instructions](https://www.nextflow.io/docs/latest/install.html) to install Nextflow.
