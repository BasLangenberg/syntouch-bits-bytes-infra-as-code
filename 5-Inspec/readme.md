# Inspec - Testing your code

With code comes automated test which you use to validate if all is working as designed. Within the configuration management space, this practice is also in place just as in "normal" software development.

We will setup a simple server and make an Inspec profile to verify it is setup correctly. In practice, this will be implemented in a CI/CD pipeline which will apply the code and then validate the setup before this is merged into an integration branch / moved to production.

## Install Inspec

First we need to install Inspec.

```SHELL
bas@devel:~$ wget https://packages.chef.io/files/stable/inspec/4.3.2/ubuntu/18.04/inspec_4.3.2-1_amd64.deb
bas@devel:~$ sudo dpkg -i inspec_4.3.2-1_amd64.deb
```

Validate the installation:

```SHELL
bas@devel:~$ which inspec
/usr/bin/inspec
bas@devel:~$ inspec
Commands:
  inspec archive PATH                # archive a profile to tar.gz (default) or zip
  inspec artifact SUBCOMMAND         # Manage InSpec Artifacts
  inspec check PATH                  # verify all tests at the specified PATH
  inspec compliance SUBCOMMAND       # Chef Compliance commands
  inspec detect                      # detect the target OS
  inspec env                         # Output shell-appropriate completion configuration
  inspec exec LOCATIONS              # run all test files at the specified LOCATIONS.
  inspec habitat SUBCOMMAND          # Manage Habitat with InSpec
  inspec help [COMMAND]              # Describe available commands or one specific command
  inspec init SUBCOMMAND             # Generate InSpec code
  inspec json PATH                   # read all tests in PATH and generate a JSON summary
  inspec plugin SUBCOMMAND           # Manage InSpec and Train plugins
  inspec shell                       # open an interactive debugging shell
  inspec supermarket SUBCOMMAND ...  # Supermarket commands
  inspec vendor PATH                 # Download all dependencies and generate a lockfile in a `vendor` directory
  inspec version                     # prints the version of this tool

Options:
  l, [--log-level=LOG_LEVEL]               # Set the log level: info (default), debug, warn, error
      [--log-location=LOG_LOCATION]        # Location to send diagnostic log messages to. (default: STDOUT or Inspec::Log.error)
      [--diagnose], [--no-diagnose]        # Show diagnostics (versions, configurations)
      [--color], [--no-color]              # Use colors in output.
      [--interactive], [--no-interactive]  # Allow or disable user interaction
      [--disable-core-plugins]             # Disable loading all plugins that are shipped in the lib/plugins directory of InSpec. Useful in development.
      [--disable-user-plugins]             # Disable loading all plugins that the user installed.
      [--chef-license=CHEF_LICENSE]        # Accept the license for this product and any contained products: accept, accept-no-persist, accept-silent
```

## Setup something to test

We will setup a pretty simple setup of an Apache webserver. We'll create it like we did in the previous tutorials.

```SHELL
bas@devel:~$ mkdir inspec_apache && cd inspec_apache
```

In your new directory, put the now well known infrastructure.tf and site.yml

infrastructure.tf

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
resource "digitalocean_droplet" "Wordpress-DB" {
  image = "ubuntu-18-04-x64"
  name = "Apache"
  region = "ams3"
  size = "2gb"
  ssh_keys = [
    "${var.ssh_fingerprint}"
  ]
}
```

site.yaml

```YAML
- hosts: all
  become: true
  roles:
    - geerlingguy.apache
```

You know the drill by now. ;-)

```SHELL
bas@devel:~/inspec_apache$ terraform init
bas@devel:~/inspec_apache$ terraform apply -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
```

Create your hosts file

```
[apache]
178.62.216.66 ansible_user=root ansible_python_interpreter=/usr/bin/python3
```

And run Ansible

```SHELL
bas@devel:~/inspec_apache$ cat terraform.tfstate | grep ipv4_add
                            "ipv4_address": "178.62.216.66",
                            "ipv4_address_private": "",
bas@devel:~/inspec_apache$ vim hosts
bas@devel:~/inspec_apache$ ansible-playbook -i hosts site.yml
```

## Write an Inspec profile

Inspec profiles look a lot like Ruby. Chef also looks a lot like ruby, it's the way their tools work. Fortunately it is still an easy to understand dsl.

Let's write a simple apache profile for inspec. Put it inside apache_profile.rb

```RUBY
# Inspec tests for apache
# Workshop version, don't take it too serious

describe package('apache2') do
  it { should be_installed }
end

describe file('/etc/apache2/apache2.conf') do
        its('group') { should eq 'root' }
        its('owner') { should eq 'root' }
        its('mode') { should eq 0644 }
end

describe port(80) do
  it { should be_listening }
  its('processes') {should include 'apache2'}
end
```

Run this profile against your remote server.

```SHELL
bas@devel:~/inspec_apache$ inspec exec apache_spec.rb -t ssh://root@178.62.216.66

Profile: tests from apache_spec.rb (tests from apache_spec.rb)
Version: (not specified)
Target:  ssh://root@178.62.216.66:22

  System Package apache2
     ✔  should be installed
  File /etc/apache2/apache2.conf
     ✔  group should eq "root"
     ✔  owner should eq "root"
     ✔  mode should eq 420
  Port 80
     ✔  should be listening
     ✔  processes should include "apache2"

Test Summary: 6 successful, 0 failures, 0 skipped
```

## Break stuff

Let's make it none complaint anymore. Login to the host you created with ssh.

```SHELLL
bas@devel:~$ ssh root@178.62.216.66
```

In /etc/apache2/ports.conf, change Listen 80 to Listen 81. Restart Apache.

```SHELL
root@Apache:/etc/apache2# systemctl restart apache2
```

Rerun your profile.

```SHELL
bas@devel:~/inspec_apache$ inspec exec apache_spec.rb -t ssh://root@178.62.216.66

Profile: tests from apache_spec.rb (tests from apache_spec.rb)
Version: (not specified)
Target:  ssh://root@178.62.216.66:22

  System Package apache2
     ✔  should be installed
  File /etc/apache2/apache2.conf
     ✔  group should eq "root"
     ✔  owner should eq "root"
     ✔  mode should eq 420
  Port 80
     ×  should be listening
     expected `Port 80.listening?` to return true, got false
     ×  processes should include "apache2"
     expected [] to include "apache2"

Test Summary: 4 successful, 2 failures, 0 skipped
```

Let's quickly fix it!

```SHELL
bas@devel:~/inspec_apache$ ansible-playbook -i hosts site.yml
bas@devel:~/inspec_apache$ inspec exec apache_spec.rb -t ssh://root@178.62.216.66

Profile: tests from apache_spec.rb (tests from apache_spec.rb)
Version: (not specified)
Target:  ssh://root@178.62.216.66:22

  System Package apache2
     ✔  should be installed
  File /etc/apache2/apache2.conf
     ✔  group should eq "root"
     ✔  owner should eq "root"
     ✔  mode should eq 420
  Port 80
     ✔  should be listening
     ✔  processes should include "apache2"

Test Summary: 6 successful, 0 failures, 0 skipped
```