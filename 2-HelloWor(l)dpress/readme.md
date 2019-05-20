# Hello Wor(l)dpress

Hello Wor(l)dpress!

In this excercise, we will install Wordpress from scratch. Not manually of course, but using Terraform and Ansible! Before starting this tutorial, make sure your development environment is setup as described in the pervious workthops.

We will setup this entire project from scratch.

## Terraform: Setup server resources

Create a new directory in your home directory. This will contain all the 'code' you need to provision this server with the software.

```shell
bas@devel:~$ mkdir ansible-wordpress
bas@devel:~$ cd ansible-wordpress
```

Create your infrastructure.tf with your favorite editor and enter the following content in it:

```HCL
# Set the variable value in *.tfvars file
# or using -var="do_token=..." CLI option
variable "do_token" {}
variable "ssh_fingerprint" {}

# Configure the DigitalOcean Provider
provider "digitalocean" {
  token = "${var.do_token}"
}

# Create Kubernetes first master
resource "digitalocean_droplet" "Wordpress" {
  image = "ubuntu-18-04-x64"
  name = "Wordpress"
  region = "ams3"
  size = "2gb"
  ssh_keys = [
    "${var.ssh_fingerprint}"
  ]
}
```

Let's explain this a bit. In the first two lines, we declare variables for your digital ocean token and your ssh fingerprint. The reason we do this, is to avoid putting this in source control. We will pass the value of these variables using the command line later.

Next, we configure Digital Ocean as our provider. During `terraform init` we will make sure the dependencies you need within Terraform are installed.

Latest part is the configuration of our server resource. This is the absolute minimum, but we should not make it more complex than absolutely necessary. ;-)

Initialize Terraform:

```SHELL
bas@devel:~/ansible-wordpress$ terraform init

Initializing provider plugins...
- Checking for available provider plugins on https://releases.hashicorp.com...
- Downloading plugin for provider "digitalocean" (1.3.0)...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.digitalocean: version = "~1.3"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Now, you can provision your node.

```shell
bas@devel:~/ansible-wordpress$ terraform apply -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + digitalocean_droplet.Wordpress
      id:                   <computed>
      backups:              "false"
      disk:                 <computed>
      image:                "ubuntu-18-04-x64"
      ipv4_address:         <computed>
      ipv4_address_private: <computed>
      ipv6:                 "false"
      ipv6_address:         <computed>
      ipv6_address_private: <computed>
      locked:               <computed>
      memory:               <computed>
      monitoring:           "false"
      name:                 "Wordpress"
      price_hourly:         <computed>
      price_monthly:        <computed>
      private_networking:   "false"
      region:               "ams3"
      resize_disk:          "true"
      size:                 "2gb"
      ssh_keys.#:           "1"
      ssh_keys.3234318775:  "a2:35:0f:16:de:80:7f:41:9b:15:0f:b1:f9:4b:2b:6a"
      status:               <computed>
      urn:                  <computed>
      vcpus:                <computed>
      volume_ids.#:         <computed>


Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

digitalocean_droplet.Wordpress: Creating...
  backups:              "" ="false"
  disk:                 "" ="<computed>"
  image:                "" ="ubuntu-18-04-x64"
  ipv4_address:         "" ="<computed>"
  ipv4_address_private: "" ="<computed>"
  ipv6:                 "" ="false"
  ipv6_address:         "" ="<computed>"
  ipv6_address_private: "" ="<computed>"
  locked:               "" ="<computed>"
  memory:               "" ="<computed>"
  monitoring:           "" ="false"
  name:                 "" ="Wordpress"
  price_hourly:         "" ="<computed>"
  price_monthly:        "" ="<computed>"
  private_networking:   "" ="false"
  region:               "" ="ams3"
  resize_disk:          "" ="true"
  size:                 "" ="2gb"
  ssh_keys.#:           "" ="1"
  ssh_keys.3234318775:  "" ="a2:35:0f:16:de:80:7f:41:9b:15:0f:b1:f9:4b:2b:6a"
  status:               "" ="<computed>"
  urn:                  "" ="<computed>"
  vcpus:                "" ="<computed>"
  volume_ids.#:         "" ="<computed>"
digitalocean_droplet.Wordpress: Still creating... (10s elapsed)
digitalocean_droplet.Wordpress: Still creating... (20s elapsed)
digitalocean_droplet.Wordpress: Still creating... (30s elapsed)
digitalocean_droplet.Wordpress: Creation complete after 35s (ID: 143785773)

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

You now have a server. In your working directory you have state file which Terraform use to compare the desired state to the actual state. It uses that to create a plan for updating the resources.

```shell
bas@devel:~/ansible-wordpress$ ll
total 20
drwxrwxr-x  3 bas bas 4096 May 13 18:20 ./
drwxr-xr-x 17 bas bas 4096 May 13 18:17 ../
drwxrwxr-x  3 bas bas 4096 May 13 18:18 .terraform/
-rw-rw-r--  1 bas bas  445 May 13 18:18 infrastructure.tf
-rw-rw-r--  1 bas bas 2248 May 13 18:20 terraform.tfstate
```

In this state file, you can find the ip address. Make a note of this, you need it in the next part.

```shell
bas@devel:~/ansible-wordpress$ cat terraform.tfstate | grep ipv4_address
                            "ipv4_address": "178.62.246.125",
                            "ipv4_address_private": "",
```

## Ansible: declare your dependencies

Let's face it, every "hello world" like application in Configuration Management starts with something like Apache Webserver. MySQL installations are already sorted out. Let's note all the dependencies for Wordpress and lean on the work of others to make our lives easy.

Dependencies:
- Apache Webserver
- PHP
- MySQL

We will use existing roles to install these components. You can look at roles as libraries. When programming Java you use these to reuse existing functionality as well.

Let's start with site.yml. This will be your main file. All dependencies are tied together here.

```YAML
- hosts: all
  become: true
  vars_files:
    - vars/main.yml
  roles:
    - geerlingguy.mysql
    - geerlingguy.apache
    - geerlingguy.php
```

The roles defined need to be installed. Ansible code bases normally have these listed in a requirements file in your roles directory. Let's define this now.

```SHELL
bas@devel:~/ansible-wordpress$ mkdir roles
bas@devel:~/ansible-wordpress$ cat > roles/requirements.yml << EOF
- src: geerlingguy.mysql
- src: geerlingguy.php
- src: geerlingguy.apache
EOF
bas@devel:~/ansible-wordpress$ ansible-galaxy install -r roles/requirements.yml
```

Now, we will define some variables. These variables are defined in the roles and will be picked up. by the play.

```SHELL
bas@devel:~/ansible-wordpress$ mkdir vars
bas@devel:~/ansible-wordpress$ vim vars/main.yml
```

```YAML
wordpress_dir: /var/www/wordpress
mysql_root_password: super-secure-password
mysql_databases:
  - name: wordpress
    encoding: latin1
    collation: latin1_general_ci
mysql_users:
  - name: wordpress
    host: "%"
    password: similarly-secure-password
    priv: "wordpress.*:ALL"
apache_listen_port: 8080
apache_vhosts:
  - {servername: "example-wordpress.com", documentroot: "{{ wordpress_dir }}/wordpress" }
wp_mysql_db: wordpress
wp_mysql_user: wordpress
wp_mysql_password: similarly-secure-password
wp_mysql_host: localhost
php_packages:
  - php
  - php-cli
  - php-common
  - php-gd
  - php-mbstring
  - php-pdo
  - php-xml
  - php-mysql
```

I already put in some variables we will use in the Wordpress specific part. There is one anti pattern in it. I wonder who will find it first. ;-)

Almost done! We need to define which host will be provisioned. You put this in a hosts file.

```SHELL
bas@devel:~/ansible-wordpress$ cat > hosts << EOF
[wordpress]
188.166.114.106 ansible_user=root ansible_python_interpreter=/usr/bin/python3
EOF
```

We connect as root. Standard Python on Ubuntu 18.04 is Python 3. Ansible looks for /usr/bin/python which does not exist.

Change the IP to the IP of the node you provisioned in the previous step. We can now provision 80% of the node, minus the actual web application.

```SHELL
bas@devel:~/ansible-wordpress$ ansible-playbook -i hosts site.yml
<KNIP>
PLAY RECAP ****************************************************************************************************************************************************
178.62.246.125             : ok=86   changed=28   unreachable=0    failed=0
```

Run the playbook again. Note the speed and the recap.

```SHELL
PLAY RECAP ****************************************************************************************************************************************************
178.62.246.125             : ok=74   changed=0    unreachable=0    failed=0
```

Well written playbooks are idempotent. When nothing is changed to the server, the code will not update the server. When writing custom modules this takes a lot of time. The author of this course did not always had the time to do this in the custom parts, but when writing production code you should always aim for this. You can then use to validate your servers configuration.

## Ansible: Fill in the gaps

Now, we will generate a new role and write our custom Wordpress code!

```SHELL
bas@devel:~/ansible-wordpress/roles$ ansible-galaxy init baslangenberg.wordpress
- baslangenberg.wordpress was created successfully
```

Change baslangenberg to your own prefix.

Edit site.yml and add your role.

```YAML
- hosts: all
  become: true
  vars_files:
    - vars/main.yml
  roles:
    - geerlingguy.mysql
    - geerlingguy.apache
    - geerlingguy.php
    - baslangenberg.wordpress
```

Now define the variables used in the playbook in roles/*.wordpress/default/main.yml

```SHELL
bas@devel:~/ansible-wordpress$ cat > roles/*.wordpress/defaults/main.yml << EOF
---
# vars file for baslangenberg.wordpress
wordpress_dir: /var/www/wordpress
wp_mysql_db: wordpress
wp_mysql_user: wordpress
wp_mysql_password: wordpress
EOF
```

We need a template for Wordpress to connect to the Wordpress database. Let's cheat.

```SHELL
bas@devel:~/ansible-wordpress$ wget https://raw.githubusercontent.com/BasLangenberg/ansible-wordpress/master/roles/baslangenberg.wordpress/templates/wp-config.php.template -O roles/*.wordpress/templates/wp-config.php.template
```

The first few lines are interesting to have a look at.

```PHP
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', '{{ wp_mysql_db}}' );

/** MySQL database username */
define( 'DB_USER', '{{ wp_mysql_user }}' );

/** MySQL database password */
define( 'DB_PASSWORD', '{{ wp_mysql_password }}' );
```

The '{{ VARIABLES }}' are variables which will be filled in when the file is parsed. The parsing of the file is setup next.

Edit roles/*wordpress/tasks/main.yml for the real logic:

```YAML
---
# Stolen from / inspired by https://dotlayer.com/how-to-use-an-ansible-playbook-to-install-wordpress/
- name: Check if wordpress is installed
  stat:
    path: /var/www/wordpress/wordpress/license.txt
  register: stat_result

- name: Download WordPress
  get_url:
    url=https://wordpress.org/latest.tar.gz
    dest=/tmp/wordpress.tar.gz
    validate_certs=no
  when: stat_result.stat.exists == False


- name: Create wordpress dir
  file:
    path: "{{ wordpress_dir }}"
    state: directory
    owner: root
    group: root
  when: stat_result.stat.exists == False

- name: Extract WordPress
  unarchive: src=/tmp/wordpress.tar.gz dest="{{ wordpress_dir }}" remote_src=yes
  become: yes
  when: stat_result.stat.exists == False

- name: Parse config file
  template:
    src: wp-config.php.template
    dest: "{{ wordpress_dir}}/wordpress/wp-config.php"
```

Let's digest this.

- First task is checking if Wordpress is already installed
  - The outcome of this decides if the other parts should run.
  - Next to the last part, that's always ran
- The second tasks downloads the latest version of Wordpress
- The third tasks creates a directory for Wordpress to live in
- The fourth will extract the tarbal into this directory
- The fifth tasks parses the config file

Save. Run the playbook.

```SHELL
<KNIP>
TASK [baslangenberg.wordpress : Check if wordpress is installed] **********************************************************************************************
ok: [178.62.246.125]

TASK [baslangenberg.wordpress : Download WordPress] ***********************************************************************************************************
changed: [178.62.246.125]

TASK [baslangenberg.wordpress : Create wordpress dir] *********************************************************************************************************
changed: [178.62.246.125]

TASK [baslangenberg.wordpress : Extract WordPress] ************************************************************************************************************
changed: [178.62.246.125]

TASK [baslangenberg.wordpress : Parse config file] ************************************************************************************************************
changed: [178.62.246.125]

PLAY RECAP ****************************************************************************************************************************************************
178.62.246.125             : ok=79   changed=4    unreachable=0    failed=0
```

Run again!

```SHELL
PLAY RECAP ****************************************************************************************************************************************************
178.62.246.125             : ok=76   changed=0    unreachable=0    failed=0
```

Still idempotent. :-)

Go to http://<IP_ADDRESS>:8080. Enjoy your work for a moment.

Then destroy everything.

```SHELL
bas@devel:~/ansible-wordpress$ terraform destroy -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
digitalocean_droplet.Wordpress: Refreshing state... (ID: 143785773)

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  - digitalocean_droplet.Wordpress


Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

digitalocean_droplet.Wordpress: Destroying... (ID: 143785773)
digitalocean_droplet.Wordpress: Still destroying... (ID: 143785773, 10s elapsed)
digitalocean_droplet.Wordpress: Destruction complete after 12s

Destroy complete! Resources: 1 destroyed.
```

In the next tutorial we will use the same code to decouple MySQL and the webservers and make them run on different hosts. This is more production like.

## Having troubles

Clone the pre-created git repository containing all the code outlined in this tutorial and apply that.

```SHELL
 $ git clone https://github.com/BasLangenberg/ansible-wordpress.git && cd ansible-wordpress && git checkout v1.0
 $ terraform apply -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
```

Update the ip adres in the hosts file as described above.

```SHELL
$ ansible-playbook -i hosts site.yml
```

After you are done, you can destroy the vm.

```SHELL
$ terraform destroy -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
```

Keep this around, you need it for the next tutorial.