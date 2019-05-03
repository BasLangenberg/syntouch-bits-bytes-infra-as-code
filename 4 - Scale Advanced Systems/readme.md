# Scaling advanced systems

After setting up a rather simple system, we will now look into how to use configuration management tools to scale an advanced clustered system. Since Kubernetes is hot and happening I took that as an example.

Clone the git repository in a directory of your choice on your server.

```
git clone https://github.com/BasLangenberg/instant-kubernetes.git
```

Run terraform. Enter yes to confirm vm creation.

```
bas@devel:~/instant-kubernetes$ terraform apply -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"

<KNIP>

Plan: 4 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

<KNIP>

digitalocean_droplet.k8s-primary-master: Still creating... (10s elapsed)
digitalocean_droplet.k8s-workers.1: Still creating... (10s elapsed)
digitalocean_droplet.k8s-workers.0: Still creating... (10s elapsed)
digitalocean_droplet.k8s-workers.2: Still creating... (10s elapsed)
digitalocean_droplet.k8s-workers.2: Still creating... (20s elapsed)
digitalocean_droplet.k8s-primary-master: Still creating... (20s elapsed)
digitalocean_droplet.k8s-workers.1: Still creating... (20s elapsed)
digitalocean_droplet.k8s-workers.0: Still creating... (20s elapsed)
digitalocean_droplet.k8s-workers.0: Still creating... (30s elapsed)
digitalocean_droplet.k8s-workers.2: Still creating... (30s elapsed)
digitalocean_droplet.k8s-primary-master: Still creating... (30s elapsed)
digitalocean_droplet.k8s-workers.1: Still creating... (30s elapsed)
digitalocean_droplet.k8s-workers[0]: Creation complete after 35s (ID: 142360837)
digitalocean_droplet.k8s-workers[1]: Creation complete after 35s (ID: 142360834)
digitalocean_droplet.k8s-workers[2]: Creation complete after 37s (ID: 142360836)
digitalocean_droplet.k8s-primary-master: Still creating... (40s elapsed)
digitalocean_droplet.k8s-primary-master: Creation complete after 49s (ID: 142360835)
```

Add your key to the ssh-agent.

```
bas@devel:~/instant-kubernetes$ eval $(ssh-agent)
Agent pid 7551
bas@devel:~/instant-kubernetes$ ssh-add ~/.ssh/id_rsa
Identity added: /home/bas/.ssh/id_rsa (/home/bas/.ssh/id_rsa)
```

Install the roles for Ansible.

```
bas@devel:~/instant-kubernetes$ ansible-galaxy install -r roles/requirements.yml
- downloading role 'docker', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-docker/archive/2.5.2.tar.gz
- extracting geerlingguy.docker to /home/bas/.ansible/roles/geerlingguy.docker
- geerlingguy.docker (2.5.2) was installed successfully
```

Now install Kubernetes. This takes a couple of minutes.

```
bas@devel:~/instant-kubernetes$ export ANSIBLE_HOST_KEY_CHECKING=FALSE
bas@devel:~/instant-kubernetes$ ansible-playbook -i hosts site.yml
```