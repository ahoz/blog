---
layout: post
title:  "Provision a server on Hetzner using Terraform and Ansible"
permalink: /provision-a-server-on-hetzner-using-terraform-and-ansible
date:   2020-06-23 08:00:00
tags: Linux Server Security Virtualisierung Terraform Ansible
---

Here is an article on how to automatically provision server on Hetzner with Terraform and Ansible.

An example configuration is prepared on [Github](https://github.com/ahoz/hcloud_terraform_ansible_provision).

The configuration creates a server with one SSH user, custom SSH config and port, IPTables firewall rules, Fail2Ban and unattended upgrades. It should be noticed, that this does not make a server highly secure. There are many measurements you can take in order to harden your server.

## Create project on Hetzner

Click on "+ NEUES PROJEKT" ("+ New Project").

![Hetzner Cloud Projects](/assets/images/01-2.png)

Nagivate to this new project.

![Hetzner Cloud Project](/assets/images/02-1.png)

Create a new API token.

![Hetzner Cloud API Tokens](/assets/images/03-1.png)

Click on create new token and store it in a safe place. It will be shown only once.

Store your SSH-Key under SSH Keys on Hetzner. Here is a good [manual](https://www.ssh.com/ssh/keygen/) on SSH keys. 
Do not forget the name you have set for the key. You'll need this later in your Terraform configuration.

## Install Terraform and Ansible

Terraform: https://learn.hashicorp.com/terraform/getting-started/install.html

Ansible: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

## Setup the project

Clone project:

{% highlight text %}
ahmet@client1:~/Desktop$ git clone git@github.com:ahoz/hcloud_terraform_ansible_provision.git
Klone nach 'hcloud_terraform_ansible_provision' ...
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (9/9), done.
Empfange Objekte: 100% (10/10), 4.03 KiB | 4.03 MiB/s, Fertig.
remote: Total 10 (delta 0), reused 10 (delta 0), pack-reused 0
{% endhighlight %}

Move variables.tf.example to variables.tf.

{% highlight text %}
ahmet@client1:~$ cd hcloud_terraform_ansible_provision/
ahmet@client1:~/hcloud_terraform_ansible_provision$ mv variables.tf.example variables.tf
{% endhighlight %}

Edit the file as explained in the [Readme](https://github.com/ahoz/hetzner-autoprovision/blob/master/Readme.md). It could like this:

{% highlight text %}
ahmet@client1:~/Desktop/hcloud_terraform_ansible_provision$ cat variables.tf
// Define your Hetzner Cloud Api Token
variable "hcloud_token" {
    type = "string"
    default = "9089hufnibdzg879802jidnbi092okwdqnjdbsi0893owdqsaodhewq"
}
// Define your Hetzner SSH Key Name
variable "ssh_key" {
    type = "string"
    default = "SSHSchluesselNameDenDuAngelegtHast"
}
// SSH root user key file path
variable "hcloud_ssh_key_local_path" {
    type = "string"
    default = "~/.ssh/id_rsa"
}
// Number of nodes you want to provision
variable "server_count" {
    type = "string"
    // Hier wird nur ein Server erstellt
    default = "1"
}
// Host Name / Host Name Prefix
variable "server_name" {
    type = "string"
    // Servername
    default = "Superserver"
}
// Root user password
variable "root_password" {
    type = "string"
    // Hier wird das Passwort des Root Nutzers geändert!
    default = "abc123"
}
// SSH user username
variable "ssh_user_username" {
    type = "string"
    // Hier ein beliebiger Nutzername. Mit diesem Nutzernamen wirst du dich später einloggen
    default = "sshnutzergeheim"
}
// SSH user password
variable "ssh_user_password" {
    type = "string"
    // Hier das Passwort zum SSH Nutzer
    default = "sshnutzergeheimpasswort:-)"
}
// SSH user keyfile path
variable "ssh_user_keyfile_path" {
    type = "string"
    // Hier noch die Möglichkeit einen anderen SSH Schlüssel für diesen Nutzer zu verwenden
    default = "~/.ssh/id_rsa_sshuser.pub"
}
// SSH port
variable "ssh_port" {
    type = "string"
    // Ein eigener SSH Port macht alles zumindest etwas sicherer.
    default = "10321"
}
// VM Location
variable "location" {
    type = "string"
    // Wir verwenden hier jetzt Nürnberg als Rechenzentrum
    default = "nbg1"
}
// VM Size
variable "server_type" {
    type = "string"
    // Die kleinstmögliche Serverinstanz von Hetzner
    default = "cx11"
}
// VM Image (Ubuntu images only)
variable "image" {
    type = "string"
    // Ubuntu 18.04
    default = "ubuntu-18.04"
}
{% endhighlight %}

## Provision servers

Next step is Terraform initialization.

{% highlight text %}
ahmet@client1:~/Desktop/hcloud_terraform_ansible_provision$ terraform init

Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "null" (hashicorp/null) 2.1.2...
- Downloading plugin for provider "hcloud" (terraform-providers/hcloud) 1.16.0...
- Downloading plugin for provider "local" (hashicorp/local) 1.4.0...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.hcloud: version = "~> 1.16"
* provider.local: version = "~> 1.4"
* provider.null: version = "~> 2.1"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

ahmet@client1:~/Desktop/hcloud_terraform_ansible_provision$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

data.hcloud_ssh_key.ssh_key: Refreshing state...

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # hcloud_server.web[0] will be created
  + resource "hcloud_server" "web" {
      + backup_window = (known after apply)
      + backups       = false
      + datacenter    = (known after apply)
      + id            = (known after apply)
      + image         = "ubuntu-18.04"
      + ipv4_address  = (known after apply)
      + ipv6_address  = (known after apply)
      + ipv6_network  = (known after apply)
      + keep_disk     = false
      + location      = "nbg1"
      + name          = "Superserver-001"
      + server_type   = "cx11"
      + ssh_keys      = [
          + "1112028",
        ]
      + status        = (known after apply)
    }

  # local_file.fileappend will be created
  + resource "local_file" "fileappend" {
      + content              = (known after apply)
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "./ansible-hosts-config"
      + id                   = (known after apply)
    }

  # null_resource.installansible[0] will be created
  + resource "null_resource" "installansible" {
      + id = (known after apply)
    }

  # null_resource.restartnodes[0] will be created
  + resource "null_resource" "restartnodes" {
      + id = (known after apply)
    }

  # null_resource.runansible[0] will be created
  + resource "null_resource" "runansible" {
      + id = (known after apply)
    }

Plan: 5 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
{% endhighlight %}

Now we can create the infrastructure.

{% highlight text %}
ahmet@client1:~/Desktop/hcloud_terraform_ansible_provision$ terraform apply
data.hcloud_ssh_key.ssh_key: Refreshing state...

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # hcloud_server.web[0] will be created
  + resource "hcloud_server" "web" {
      + backup_window = (known after apply)
      + backups       = false
      + datacenter    = (known after apply)
      + id            = (known after apply)
      + image         = "ubuntu-18.04"
      + ipv4_address  = (known after apply)
      + ipv6_address  = (known after apply)
      + ipv6_network  = (known after apply)
      + keep_disk     = false
      + location      = "nbg1"
      + name          = "Superserver-001"
      + server_type   = "cx11"
      + ssh_keys      = [
          + "1112028",
        ]
      + status        = (known after apply)
    }

  # local_file.fileappend will be created
  + resource "local_file" "fileappend" {
      + content              = (known after apply)
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "./ansible-hosts-config"
      + id                   = (known after apply)
    }

  # null_resource.installansible[0] will be created
  + resource "null_resource" "installansible" {
      + id = (known after apply)
    }

  # null_resource.restartnodes[0] will be created
  + resource "null_resource" "restartnodes" {
      + id = (known after apply)
    }

  # null_resource.runansible[0] will be created
  + resource "null_resource" "runansible" {
      + id = (known after apply)
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

hcloud_server.web[0]: Creating...
hcloud_server.web[0]: Creation complete after 10s [id=3179189]
null_resource.installansible[0]: Creating...
local_file.fileappend: Creating...
local_file.fileappend: Creation complete after 0s [id=9c1b97d3fbb3043995b3b29c342b0c39b233c604]
null_resource.installansible[0]: Provisioning with 'remote-exec'...
null_resource.installansible[0] (remote-exec): Connecting to remote host via SSH...
null_resource.installansible[0] (remote-exec):   Host: 212.17.145.9
null_resource.installansible[0] (remote-exec):   User: root
null_resource.installansible[0] (remote-exec):   Password: false
null_resource.installansible[0] (remote-exec):   Private key: true
null_resource.installansible[0] (remote-exec):   Certificate: false
null_resource.installansible[0] (remote-exec):   SSH Agent: true
null_resource.installansible[0] (remote-exec):   Checking Host Key: false
null_resource.installansible[0]: Still creating... [10s elapsed]
null_resource.installansible[0] (remote-exec): Connecting to remote host via SSH...
null_resource.installansible[0] (remote-exec):   Host: 212.17.145.9
null_resource.installansible[0] (remote-exec):   User: root
null_resource.installansible[0] (remote-exec):   Password: false
null_resource.installansible[0] (remote-exec):   Private key: true
null_resource.installansible[0] (remote-exec):   Certificate: false
null_resource.installansible[0] (remote-exec):   SSH Agent: true
null_resource.installansible[0] (remote-exec):   Checking Host Key: false
null_resource.installansible[0] (remote-exec): Connecting to remote host via SSH...
null_resource.installansible[0] (remote-exec):   Host: 212.17.145.9
null_resource.installansible[0] (remote-exec):   User: root
null_resource.installansible[0] (remote-exec):   Password: false
null_resource.installansible[0] (remote-exec):   Private key: true
null_resource.installansible[0] (remote-exec):   Certificate: false
null_resource.installansible[0] (remote-exec):   SSH Agent: true
null_resource.installansible[0] (remote-exec):   Checking Host Key: false
null_resource.installansible[0] (remote-exec): Connected!
null_resource.installansible[0] (remote-exec):

<INSTALLATION VON UPDATES UND ANSIBLE>

null_resource.installansible[0]: Creation complete after 1m5s [id=3346375517487291883]
null_resource.runansible[0]: Creating...
null_resource.runansible[0]: Provisioning with 'local-exec'...
null_resource.runansible[0] (local-exec): Executing: ["/bin/sh" "-c" "ansible-playbook -i ansible-hosts-config ansible-dist.yml --extra-vars 'ssh_user_username=sshnutzergeheim ssh_user_password=sshnutzergeheim ssh_port=10321 ssh_user_keyfile_path=~/.ssh/id_rsa_sshuser.pub root_password=abc123'"]
null_resource.runansible[0] (local-exec): [DEPRECATION WARNING]: The TRANSFORM_INVALID_GROUP_CHARS settings is set to
null_resource.runansible[0] (local-exec): allow bad characters in group names by default, this will change, but still be
null_resource.runansible[0] (local-exec): user configurable on deprecation. This feature will be removed in version 2.10.
null_resource.runansible[0] (local-exec):  Deprecation warnings can be disabled by setting deprecation_warnings=False in
null_resource.runansible[0] (local-exec): ansible.cfg.
null_resource.runansible[0] (local-exec):  [WARNING]: Invalid characters were found in group names but not replaced, use
null_resource.runansible[0] (local-exec): -vvvv to see details

null_resource.runansible[0] (local-exec): PLAY [hetzner-cloud] ***********************************************************

null_resource.runansible[0] (local-exec): TASK [Gathering Facts] *********************************************************
null_resource.runansible[0] (local-exec): ok: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Update] ******************************************************************
null_resource.runansible[0]: Still creating... [10s elapsed]
null_resource.runansible[0]: Still creating... [20s elapsed]
null_resource.runansible[0]: Still creating... [30s elapsed]
null_resource.runansible[0]: Still creating... [40s elapsed]
null_resource.runansible[0]: Still creating... [50s elapsed]
null_resource.runansible[0]: Still creating... [1m0s elapsed]
null_resource.runansible[0]: Still creating... [1m10s elapsed]
null_resource.runansible[0]: Still creating... [1m20s elapsed]
null_resource.runansible[0] (local-exec):  [WARNING]: Updating cache and auto-installing missing dependency: python-apt
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Install IPTables Persistent] *********************************************
null_resource.runansible[0]: Still creating... [1m30s elapsed]
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [SSH group] ***************************************************************
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [SSH user] ****************************************************************
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [SSH user key] ************************************************************
null_resource.runansible[0]: Still creating... [1m40s elapsed]
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [SSH Config update] *******************************************************
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Iptables IPv4 rules] *****************************************************
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Iptables IPv6 rules] *****************************************************
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Root password] ***********************************************************
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Update SSH port in ssh config] *******************************************
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Update SSH port in iptables rules (IPv4)] ********************************
null_resource.runansible[0]: Still creating... [1m50s elapsed]
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Update SSH port in iptables rules (IPv6)] ********************************
null_resource.runansible[0] (local-exec): ok: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Create fail2ban config] **************************************************
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Enable fail2ban ssh] *****************************************************
null_resource.runansible[0] (local-exec): ok: [212.17.145.9]

null_resource.runansible[0] (local-exec): TASK [Enable unattended upgrades] **********************************************
null_resource.runansible[0] (local-exec): changed: [212.17.145.9]

null_resource.runansible[0] (local-exec): PLAY RECAP *********************************************************************
null_resource.runansible[0] (local-exec): 212.17.145.9             : ok=16   changed=13   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

null_resource.runansible[0]: Creation complete after 1m55s [id=1017434632877430811]
null_resource.restartnodes[0]: Creating...
null_resource.restartnodes[0]: Provisioning with 'remote-exec'...
null_resource.restartnodes[0] (remote-exec): Connecting to remote host via SSH...
null_resource.restartnodes[0] (remote-exec):   Host: 212.17.145.9
null_resource.restartnodes[0] (remote-exec):   User: root
null_resource.restartnodes[0] (remote-exec):   Password: false
null_resource.restartnodes[0] (remote-exec):   Private key: true
null_resource.restartnodes[0] (remote-exec):   Certificate: false
null_resource.restartnodes[0] (remote-exec):   SSH Agent: true
null_resource.restartnodes[0] (remote-exec):   Checking Host Key: false
null_resource.restartnodes[0] (remote-exec): Connected!
null_resource.restartnodes[0]: Creation complete after 1s [id=4223191552662426200]

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

Outputs:

Done =
 If ansible did not have printed any errors, then all nodes are provisioned successfully :-)
Do not forget to delete variables.tf, since it contains sensitive information!
ip = 212.17.145.9
{% endhighlight %}