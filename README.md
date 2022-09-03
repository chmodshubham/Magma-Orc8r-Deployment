# Deploy Magma Orchestrator

Deploying magma orchestrator using ansible playbook(script).

## Prerequisite

1. Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu).

```bash
sudo apt purge ansible
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

2. Install dependant collections.

```bash
ansible-galaxy collection install shubhamtatvamasi.magma
```

3. It is recommended to deploy the orc8r in a new VM so that it won't interfere with the installed applications in your system.

## Installation

### Copy SSH Key

1. Login to the VM.

```bash
ssh ubuntu@192.168.5.192
```

- This will add `192.168.5.192` to the list of known hosts.

2. Then on the other terminal, copy the public key to remote-host using `ssh-copy-id` command.

```bash
ssh-keygen -R 192.168.5.192
ssh-copy-id ubuntu@192.168.5.192
```
- `ssh-keygen` can create keys for use by SSH protocol. `ssh-copy-id` installs an SSH key on the VM as an authorized key. Its purpose is to provision access without requiring a password for each login. This will facilitates automated, passwordless logins and single sign-on using the SSH protocol.

- You can check this by login again to the VM. This time, no password is required while login.

### Clone Repository & Update Values

Clone [ShubhamTatvamasi/magma-galaxy](https://github.com/ShubhamTatvamasi/magma-galaxy) repository in your local system.

Update `hosts` field with the VM's IP address and change the `orc8r_domain` name with your orc8r name in `hosts.yml`.

```bash
git clone https://github.com/ShubhamTatvamasi/magma-galaxy.git
cd magma-galaxy/
vim hosts.yml
```

Uncomment the commented part if deploying in AWS.

![image](https://user-images.githubusercontent.com/97805339/181386376-e1fe1ea8-a345-4f72-8c2b-27712dad0428.png)

### Deploy Magma Orc8r

`ansible-playbook` is a list of tasks that automatically execute against all the hosts that are included in the ansible inventory.

```bash
ansible-playbook deploy-orc8r.yml
```

> Note: Deployment takes 10-20 min and apart from this, it needs extra 5-10 min to start all the magma services. You can check if the pods are running or not by login into your VM and run this command `kubectl get pods -A`.

### Update DNS Values

After deployment is done, DNS Values will appear at the end.

![image](https://user-images.githubusercontent.com/97805339/181389335-a325222c-7bfb-4540-a886-8e33eac716ea.png)

Copy the message part and append it at the end of `/etc/hosts`.

```bash
cd
sudo vim /etc/hosts
```

Also update the 2nd message:

`192.168.5.192 *.nms.galaxy.shubhamkumar89.com`<br>
to <br>
`192.168.5.185 master.nms.galaxy.shubhamaditya.com` <br>
`192.168.5.185 magma-test.nms.galaxy.shubhamaditya.com`

Explaination of this step is [here](https://docs.magmacore.org/docs/nms/deploy_config).

![image](https://user-images.githubusercontent.com/97805339/181390182-36b44e9d-674d-401d-bcad-0e9fa73ea315.png)

### Create New User

When we deploy the NMS for the first time, we need to create a new user that has access to the master and magma-test organization.

> Note: Run the below command in the VM.

```bash
ORC_POD=$(kubectl -n orc8r get pod -l app.kubernetes.io/component=orchestrator -o jsonpath='{.items[0].metadata.name}')

kubectl -n orc8r exec -it ${ORC_POD} -- envdir /var/opt/magma/envdir /var/opt/magma/bin/accessc \
  add-existing -admin -cert /var/opt/magma/certs/admin_operator.pem admin_operator

NMS_POD=$(kubectl -n orc8r get pod -l app.kubernetes.io/component=magmalte -o jsonpath='{.items[0].metadata.name}')

kubectl -n orc8r exec -it ${NMS_POD} -- yarn setAdminPassword magma-test admin admin
kubectl -n orc8r exec -it ${NMS_POD} -- yarn setAdminPassword master admin admin
```

### Login:

Login to the NMS through URL:

magma-test.nms.yourdomain.com <br>
master.nms.yourdomain.com <br>

e.g.
https://master.nms.galaxy.shubhamkumar89.com, <br>
https://magma-test.nms.galaxy.shubhamkumar89.com

<b> Email: admin<br>
Password: admin<b>

![image](https://user-images.githubusercontent.com/97805339/181392698-f0440338-6484-4e93-b85b-7a4b8259255a.png)
