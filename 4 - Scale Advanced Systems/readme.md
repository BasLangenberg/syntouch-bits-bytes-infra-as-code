# Scaling advanced systems

After setting up a rather simple system, we will now look into how to use configuration management tools to scale an advanced clustered system. Since Kubernetes is hot and happening I took that as an example.

Clone the git repository in a directory of your choice on your server.

```SHELL
git clone https://github.com/BasLangenberg/instant-kubernetes.git
```

Run terraform. Enter yes to confirm vm creation.

```SHELL
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

```SHELL
bas@devel:~/instant-kubernetes$ eval $(ssh-agent)
Agent pid 7551
bas@devel:~/instant-kubernetes$ ssh-add ~/.ssh/id_rsa
Identity added: /home/bas/.ssh/id_rsa (/home/bas/.ssh/id_rsa)
```

Install the roles for Ansible.

```SHELL
bas@devel:~/instant-kubernetes$ ansible-galaxy install -r roles/requirements.yml
- downloading role 'docker', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-docker/archive/2.5.2.tar.gz
- extracting geerlingguy.docker to /home/bas/.ansible/roles/geerlingguy.docker
- geerlingguy.docker (2.5.2) was installed successfully
```

Let's generate the host file this time.

```SHELL
bas@verify:~/instant-kubernetes$ python generate_hosts_file.py > hosts
```

Now install Kubernetes. This takes a couple of minutes.

```SHELL
bas@devel:~/instant-kubernetes$ export ANSIBLE_HOST_KEY_CHECKING=FALSE
bas@devel:~/instant-kubernetes$ ansible-playbook -i hosts site.yml
```

When this is running, there will be a moment Ansible waits for the master node to be bootstrapped completely.

```SHELL
TASK [kubernetes : verify if master is ok before continuing!] *************************
Pausing for 300 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
[kubernetes : verify if master is ok before continuing!]
Make sure the master node is done provisioning, verify this. The playbook will wait for 5 minutes:
Press 'C' to continue the play or 'A' to abort
```

Open a new connection, enter kubectl get nodes. This is all setup on your development node by Ansible

```SHELL
bas@devel:~/instant-kubernetes$ kubectl get nodes
NAME                 STATUS   ROLES    AGE   VERSION
k8s-primary-master   Ready    master   19m   v1.14.1
```

When the master is ready, enter ctrl+c, c to make Ansible bootstrap te worker nodes. After thats done, more will be in the output of kubectl get nodes.

```SHELL
bas@devel:~/instant-kubernetes$ kubectl get nodes
NAME                 STATUS   ROLES    AGE   VERSION
k8s-primary-master   Ready    master   19m   v1.14.1
k8s-worker-0         Ready    <none>   18m   v1.14.1
k8s-worker-1         Ready    <none>   18m   v1.14.1
k8s-worker-2         Ready    <none>   18m   v1.14.1
```

Now we will scale the system by adding more workers. Edit infrastructure 

- Edit infrastructure.tf. Change the amount of nodes under workers to 5
- Run Terraform
- regenerate your hosts file
- Run Ansible
- Check your nodes

```
bas@devel:~/instant-kubernetes$ kubectl get nodes
NAME                 STATUS   ROLES    AGE   VERSION
k8s-primary-master   Ready    master   19m   v1.14.1
k8s-worker-0         Ready    <none>   18m   v1.14.1
k8s-worker-1         Ready    <none>   18m   v1.14.1
k8s-worker-2         Ready    <none>   18m   v1.14.1
k8s-worker-3         Ready    <none>   13m   v1.14.1
k8s-worker-4         Ready    <none>   13m   v1.14.1
```

This demonstrates how you can easily use Configuration Management tools to scale up distributed systems while making sure the configuration is uniform on all nodes.

Clean up:
```
terraform destroy -var "do_token=${DO_ACCESS_TOKEN}" -var "ssh_fingerprint=${DO_SSH_FINGERPRINT}"
```