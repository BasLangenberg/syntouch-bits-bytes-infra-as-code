# Wordpress, next level

Now, we will continue our Wordpress adventure. As proper sysops, we will move the database off to a different host. We all know how busy webapps can be...

The objective of this tutorial is to show how you can leverage previous work when architectural demands change.

## Modify Terraform

First we need to update our Terraform config. We now want 2 machines instead of one. Edit infrastructure.tf

```YAML
# Set the variable value in *.tfvars file
# or using -var="do_token=..." CLI option
variable "do_token" {}
variable "ssh_fingerprint" {}

# Configure the DigitalOcean Provider
provider "digitalocean" {
  token = "${var.do_token}"
}

# Create Kubernetes first master
resource "digitalocean_droplet" "Wordpress-DB" {
  image = "ubuntu-18-04-x64"
  name = "Wordpress-db"
  region = "ams3"
  size = "2gb"
  ssh_keys = [
    "${var.ssh_fingerprint}"
  ]
}

resource "digitalocean_droplet" "Wordpress-App" {
  image = "ubuntu-18-04-x64"
  name = "Wordpress-app"
  region = "ams3"
  size = "2gb"
  ssh_keys = [
    "${var.ssh_fingerprint}"
  ]
}
```

Easy enough, now provision!

```SHELL
bas@devel:~/ansible-wordpress$ terraform apply -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:
<KNIP>
Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
<KNIP>
digitalocean_droplet.Wordpress-App: Still creating... (50s elapsed)
digitalocean_droplet.Wordpress-DB: Still creating... (50s elapsed)
digitalocean_droplet.Wordpress-DB: Creation complete after 55s (ID: 144065640)
digitalocean_droplet.Wordpress-App: Still creating... (1m0s elapsed)
digitalocean_droplet.Wordpress-App: Still creating... (1m10s elapsed)
digitalocean_droplet.Wordpress-App: Creation complete after 1m17s (ID: 144065635)

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Look in terraform.tfstate. There are two IP's now. Make a note which IP belongs to which server and update your hosts file.

```
[wordpress-app]
104.248.89.179 ansible_user=root ansible_python_interpreter=/usr/bin/python3

[wordpress-db]
134.209.80.41 ansible_user=root ansible_python_interpreter=/usr/bin/python3
```

## Change some variables around

Now the infrastructure is provisioned, we can start modifying the play.

Update the template. Make sure the DB_HOST line looks like below.

```YAML
define( 'DB_HOST', '{{ wp_mysql_host }}' );
```

Now add a variable to the variable file. (vars/main.yml)

```YAML
wp_mysql_host: <IP_OF_YOUR_DB_HOST>
```

This is a small improvement with the previous version. If you want to do the one host setup again, only the variable needs to be changed back to localhost to make it work.

## Change the way we play

The play will be split up in two runs instead of one. Functionally nothing changes that much.

```YAML
- hosts: wordpress-db
  become: true
  vars_files:
    - vars/main.yml
  roles:
    - geerlingguy.mysql

- hosts: wordpress-app
  become: true
  vars_files:
    - vars/main.yml
  roles:
    - geerlingguy.apache
    - geerlingguy.php
    - baslangenberg.wordpress
```

Now run the play. Go to the ip address of your Wordpress application host. You should see the same screen.

```SHELL
bas@devel:~/ansible-wordpress$ ansible-playbook -i hosts site.yml
```

Again run destroy to prevent charges.

```SHELL
bas@devel:~/ansible-wordpress$ terraform destroy -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
```

## Having issues?

Clone the pre-created git repository containing all the code outlined in this tutorial and apply that.

```SHELL
 $ git clone https://github.com/BasLangenberg/ansible-wordpress.git && cd ansible-wordpress && git checkout v2.0
 $ terraform apply -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
```

Update the ip adresses in the hosts file as described above.

```SHELL
$ ansible-playbook -i hosts site.yml
```

After you are done, you can destroy the vm.

```SHELL
$ terraform destroy -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
```