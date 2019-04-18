# Chapter 1: Provision your development node

[More information about cloud-init](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting)

It's the chicken and the egg all over again. All of the tools used in this workshop are Linux based, but not everybody has access to a Linux machine. It would be very strange to setup a Linux machine manually on this event! So we will automate it as much as possible.

```
#cloud-config
users:
  - name: your-name-here
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa YOUR-KEY your-name@syntouch.nl
packages:
 - terraform
 - tmux
 - vim
 - ansible
package_upgrade: true
power_state:
  timeout: 120
  delay: "+5"
  message: Reboot after setup
  mode: reboot
```

Full sample:

```
#cloud-config
users:
  - name: bas
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDjAZ5g2pMWXNxk31kdSDJdfduIxZhRU2DgeCOnD5Yo18tqDGISbDpuVhJwClFvnK1dKt7/hTi67Xrm9Wl/QWZzdDymARCe4mxOvGEHKg+oUVa8IbzL2gDCdtLoSfnq+0KX+qyZbfrUJ7Kxv+Bc/XBt13WaWkLtFuGYQ0kPSuihwKI87MoDPpLkXvCdg4wbvziGJ95dUVxKmwMMdfZt+Zic2v6c+GuQC/fK5AkRd0MbwYWTT1Pxo6mXAdn3T62PV6Z6/ruIVjhTcMfx3FuGVk8EhvYCE9nuT/sAmGfyyjLwiU/xRXFeHBEPjFX+Hjva/wsglY9saw4DAiYfdgJvKHsx AzureAD+BasLangenberg@DESKTOP-RFVONSL
package_upgrade: true
packages:
 - tmux
 - vim
 - ansible
runcmd:
  - [ snap, install, terraform ]
power_state:
  timeout: 120
  delay: "now"
  message: Reboot after setup
  mode: reboot
```
after a couple of minutes you should be able to login with your username and own private key.

 Figure out what happened by checking /var/log/cloud-init-output.log. It details what cloud-init did to the server after the initial provision form the Digital Ocean template.