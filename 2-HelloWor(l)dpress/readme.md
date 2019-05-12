# Hello Wor(l)dpress

Hello Wor(l)dpress!

In this excercise, we will install Wordpress from scratch. Not manually of course, but using Terraform and Ansible! Before starting this tutorial, make sure your development environment is setup as described in the pervious workthops.

We will setup this entire project from scratch.

## Setup server resources with Terraform

Create a new directory in your home directory. The 

## Having troubles

Clone the pre-created git repository containing all the code outlined in this tutorial and apply that.

```shell
 $ git clone https://github.com/BasLangenberg/ansible-wordpress.git && cd ansible-wordpress
 $ terraform apply -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
```

Update the ip adres in the hosts file as described above.

```shell
$ ansible-playbook -i hosts site.yml
```

After you are done, you can destroy the vm.

```shell
$ terraform destroy -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
```

Keep this around, you need it for the next tutorial.