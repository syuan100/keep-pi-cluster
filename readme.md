# Redundant KEEP Staking on a Raspberry Pi Cluster

The purpose of this project is to provide a (relatively) easy way to deploy a fault-tolerant KEEP staking cluster on Raspberry Pi 4's using Ansible. Familiarity with Kubernetes, or at least a willingness to learn, is recommended so that you can use the `kubectl` commands to troubleshoot your deployments if anything goes sideways.

#### What does this project contain?
- Full hardware setup instructions for a Raspberry Pi cluster
- Single-line command for a self-healing deployment of geth, beacon, and validator nodes
- A single variables file `group_vars/all.yml` that controls your deployment
- Additional playbooks to update your deployment

#### Open source to the core

Please feel free to fork, make PRs, and open issues! These scripts can easily be adapted for other architectures and applications.

---

#### So who exactly is this for...
- People who want to create a cost effective, yet redudant, staking setup
- People who want to learn a bit about Keep, Ansible, Kubernetes, and GlusterFS
- People who are just curious to see how Kubernetes can be used to self-heal deployments

#### ⚠️ CAUTION ⚠️
**Your Ether/KEEP is your responsibility.** Reading the code and understanding how this deployment works BEFORE committing any tokens (test or otherwise) will increase your likelyhood of success in troubleshooting. While this project was created for the Ropsten testnet, it can easily be adapted for mainnet usage in the future, if you choose to take that risk. It is recommended that for a mainnet deployment you increase storage capacity and invest in redudant backup power supplies, as well as a backup source of internet connection, among other redundancies.

**THIS GUIDE IS PURELY INFORMATIONAL. PLEASE DO YOUR RESEARCH IF YOU WISH TO IMPLEMENT THIS WITH REAL TOKENS**

#### A small shout out to the pre-made ansible playbooks that I used:

- [Ubuntu Ansible playbooks by Digital Ocean](https://github.com/do-community/ansible-playbooks)
- [k3s-ansible by Rancher](https://github.com/rancher/k3s-ansible)
- [gluster-ansible by Gluster](https://github.com/gluster/gluster-ansible)

## Hardware
 - 3+ Raspberry Pi 4s with at least 2GB of RAM. ([Canakit](https://www.canakit.com/raspberry-pi-4-2gb.html))
 - 3+ MicroSD cards ([SanDisk Extreme](https://www.amazon.com/SanDisk-Ultra-microSDXC-Memory-Adapter/dp/B073JWXGNT/))
 - 5-port gigabit unmanaged switch ([Linksys one I bought](https://www.amazon.com/Linksys-Business-LGS105-Unmanaged-Enclosure/dp/B00FV12VSW/))
 - Ethernet cables ([Amazon](https://www.amazon.com/gp/product/B01J8KFTB2/))
 - 3+ USB 3.0 SSD (external storage for each pi)
 - (Optional - But highly recommended) USB Wall Charger ([Anker](https://www.amazon.com/gp/product/B00P936188/))
    - Also get: USB-A to USB-C cables ([Amazon](https://www.amazon.com/gp/product/B07K24TKL6/))
 - (Optional) Raspberry Pi tower enclosure ([Amazon one with built in fans!](https://www.amazon.com/gp/product/B07MW24S61/))

## Raspberry Pi 4 setup

### Flash the OS
- Download the Ubuntu 18.04 64-bit image for Raspberry Pi 4 ([Link](https://ubuntu.com/download/raspberry-pi))
- Download Balena Etcher ([Link](https://www.balena.io/etcher/))
- Open up Balena Etcher and follow the instructions to flash the Ubuntu image to your microSD card

*Note: Please do not use Raspberry Pi Imager. For reasons unknown, using it caused me to be unable to properly SSH into the Raspberry Pi.*

### Enable SSH
- Once flashed, unplug and re-plug the microSD card back into your machine
- Add an emtpy file named `ssh` on the `system-boot` partition (Note: Some images may already have the file there upon flashing)

You can now insert the microSD card into the Raspberry Pis.

### Connect everyting to your network

Your setup should look something like this:

[ Modem * ]---[ Router * ]---[ Switch ]---[ Raspberry Pis ]

_* You should already have these_

#### ⚠️ SECURTIY WARNING ⚠️
**This guide assumes you have taken all the proper and necessary steps to secure your network traffic through the router. IMPROPER CONFIGURATION OR CARELESS PORT FORWARDING/OPENING CAN RESULT IN YOUR DEVICES GETTING HACKED AND POTENTIALLY LOSING YOUR STAKED ETHER.**

Check out the page below for tips on how to test your router security
[https://www.routersecurity.org/testrouter.php](https://www.routersecurity.org/testrouter.php)

### Write down the local IP address for each Pi
Once you've properly connected everyting and the Raspberry Pis have completed booting, log into your router to see what each Pi's IP address is.

You may want to setup IP reservations for each Pi so that you don't need to check everytime you reboot or unplug your Pi. Most routers support this through their UI, or [read this article for general instructions on how to do that](https://lifehacker.com/how-to-set-up-dhcp-reservations-and-never-check-an-ip-5822605).

✏️ Take note of each IP address, you will need it for the Ansible step later.

### SSH setup
1. From your host machine, open a terminal and type:
  
   `ssh ubuntu@xxx.xxx.x.x` (replace the x's with the IP address of the Pi)

2. The default password is `ubuntu`. Upon successfully logging in you will be prompted to update the password.

    *Note: You will only be using password login during this step. For the rest of the guide, we will setup SSH keys and disable password login to increase security.*

3. Go back to your host machine and type this command to see if you have a set of SSH keys already:
   
   `ls -al ~/.ssh` 
   
   If you see something like `id_rsa.pub` as one of the files then you can skip step 4.

4. If you don't have an SSH key already for your host machine, generate an SSH key:
    
    `ssh-keygen`
    
    Follow the instructions. Note: This process will be faster if you don't use a passphrase.
    - If you DO use a passphrase on your key, follow the instructions [here](https://docs.ansible.com/ansible/latest/user_guide/connection_details.html#ssh-key-setup) for how to use `ssh-agent` to store your passphrase.

5. Copy the SSH keys to your Raspberry Pis:
   
   `ssh-copy-id ubuntu@xxx.xxx.x.x`
   
   Once your keys are successfully copied, you should be able to login without using the password:
   - Test it out! `ssh ubuntu@xxx.xxx.x.x`
   - When you are **SURE** you can login to each Pi using SSH without a password, move on to the next section.


## Ansible setup
1. Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu)
2. Install a few dependencies for the Ansible modules we are using:
   
   `pip install jmespath` or `apt-get install python-jmespath`

## Replicated storage setup
*Note: Attaching a USB stick or SSD drive to your Raspberry Pi will be easier and more performant than partitioning a portion of your SD card.*

1. Attach your SSD drive to the upper blue USB 3.0 port of each Pi

2. `ssh` into your Pis and check what the device path is by running this command:
   
   `lsblk`
   
   You should see an output similar to this:
   
   ```
   NAME                             MAJ:MIN   RM   SIZE RO TYPE MOUNTPOINT
    sda                                 8:0    1  58.4G  0 disk
    └─sda1                              8:1    1  58.4G  0 part
    mmcblk0                           179:0    0 238.3G  0 disk
    ├─mmcblk0p1                       179:1    0   256M  0 part /boot/firmware
    └─mmcblk0p2                       179:2    0   238G  0 part /
   ```
   
   On *most* Pis, your USB SSD will show up as `sda1`. This means your device path will be `/dev/sda1`. If you already have a USB port occupied, it's possible that your path may be `sdb1` or `sdc1`. The important part is that it follows the convention of `sdx#`.
   
   ✏️Take note of the device path, you'll need it for the Ansible step later.


## Ansible playbook setup

- In the repository, remove the `.example` extension from these files:
  - `hosts.ini.example`
  - `group_vars/all.yml.example`
- Edit the `hosts.ini` file: You will need an IP for one main "master" Pi node and the rest will be "worker" node Pis

  ```
  [master]
  ; Put the IP address for your main Kubernetes server
  xxx.xxx.xx.xxx

  [nodes]
  ; Put the IP addresses for the rest of the nodes
  xxx.xxx.xx.xxx
  xxx.xxx.xx.xxx
  xxx.xxx.xx.xxx
  ```

- Edit the file in `group_vars/all.yml` to suit your needs. Head to the `group_vars` directory to see the README of all the details.

  Here is a list of variables that MUST be changed:
  
  ```
  # Use the endpoints provided from your Infura account
  eth1_endpoint: https://ropsten.infura.io/v3/XXXXXXXXXXXXXXXXXXXXXXXX
  ethereum_url: wss://ropsten.infura.io/ws/v3/XXXXXXXXXXXXXXXXXXXXXXXX
  
  # storage_path will depend on how you plugged in your device. Use the command lsblk while logged into your Raspberry Pi to see what to change this value to.
  storage_path: /dev/sda1
  
  # Change storage_size to be LESS than the amount of external storage you have plugged into one Pi
  storage_size: 30
  
  # Change to your account address authorized to for the random beacon
  beacon_account_address: "0xAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"
  
  # The name for your keystore file exactly
  beacon_keystore_filename: UTC--2020-08-29T17-45-22.740497000Z--AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
  
  # The password used to secure your keystore file
  beacon_account_password: yourpasswordhere
  
  # Your public IP address for P2P discovery
  beacon_announced_addresses: '"/ip4/xxx.xxx.xxx/tcp/3919"'
  ```

## Keystore setup

Ansible will copy the keystore files from the `keystore` directory over to your cluster. Before running the playbooks, please copy and paste your keystore files into that directory.

## Run the setup playbook

While at the root level of the repository, run this command:

`ansible-playbook -i hosts.ini playbooks/initial_setup.yml`

You will see the playbook run and you can monitor each step as it completes. The playbook end to end takes about 10 minutes to run.

If you are running into errors, you can run the above command with the `-v`, `-vv`, or `-vvv` flag to see more details about each step as it runs.

If you've deployed successfully, SSH into your master node and run this command:

`sudo kubectl get all -o wide`

You should see something like this:

```
NAME                                      READY   STATUS    RESTARTS   AGE    IP           NODE     NOMINATED NODE   READINESS GATES
pod/keep-ecdsa-5f8b7f5dcb-6885b           1/1     Running   1          148m   10.42.1.11   rpi-02   <none>           <none>
pod/keep-random-beacon-7c87cc5db5-hgqcg   1/1     Running   1          147m   10.42.2.22   rpi-03   <none>           <none>

NAME                         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE    SELECTOR
service/kubernetes           ClusterIP      10.43.0.1       <none>        443/TCP                         2d7h   <none>
service/keep-random-beacon   LoadBalancer   10.43.188.228   <pending>     3919:30011/TCP                  23h    app=keep-random-beacon
service/keep-ecdsa           LoadBalancer   10.43.120.140   <pending>     3920:30012/TCP                  20h    app=keep-ecdsa

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS           IMAGES                                SELECTOR
deployment.apps/keep-ecdsa           1/1     1            1           20h    keep-ecdsa           syuan100/keep-ecdsa:v1.1.1-rc_arm64   app=keep-ecdsa
deployment.apps/keep-random-beacon   1/1     1            1           23h    keep-random-beacon   syuan100/keep-core:v1.3.0-rc_arm64    app=keep-random-beacon

NAME                                            DESIRED   CURRENT   READY   AGE     CONTAINERS           IMAGES                                SELECTOR
replicaset.apps/keep-ecdsa-5f8b7f5dcb           1         1         1       148m    keep-ecdsa           syuan100/keep-ecdsa:v1.1.1-rc_arm64   app=keep-ecdsa,pod-template-hash=5f8b7f5dcb
replicaset.apps/keep-random-beacon-7c87cc5db5   1         1         1       147m    keep-random-beacon   syuan100/keep-core:v1.3.0-rc_arm64    app=keep-random-beacon,pod-template-hash=7c87cc5db5
```

### Useful `kubectl` commands

#### Check logs for each deployment

`sudo kubectl logs -l app=app-name-here -f --since=10m`

Replace `app-name-here` with the appropriate app label for the deployment you want to inspect (i.e. `keep-random-beacon`, `keep-ecdsa`)

#### Execute commands in a pod

`sudo kubectl -it exec keep-random-beacon-7c87cc5db5-hgqcg sh`

Replace `keep-random-beacon-7c87cc5db5-hgqcg` with whatever pod you wish to login to. You can also replace `sh` with `/bin/bash` if you prefer.


### Other playbooks

Run all of the below with `ansible-playbook -i hosts.ini`

`1_ubuntu_playbook.yml`: Initial setup for the Raspberry Pi - including disabling password SSH login, updating host names, and updating apt.

`2_k3s_playbook.yml`: Setup kubernetes - including Pi specific settings, downloading and installing master and worker nodes, and setting hard and soft pod eviction limits.

`3_glusterfs_playbook.yml` : Setup replicated storage - including formatting the specified drive, creating an LVM group, installing and starting a GlusterFS volume.

`4_services_playbook.yml` : Run containers and setup services - including setting up GlusterFS persistent volumes, MetalLB load balancer, copying application specific config files, and deploying KEEP staking nodes.

In addition, there are utility playbooks under playbooks/utils :

`remove_glusterfs.yml` : Removes GlusterFS and deletes all content on drives. Useful if you are having issues with replicated storage and need to start fresh.

`update_deployment.yml` : Similar to the services playbook, except it removes all relevant services before deploying new configuration.

`update_kubelet_config.yml` : If you wish to tweak kubelet parameters, you can run this script afterwards to apply the changes.

---

## Further improvements for consideration
- Prometheus + Grafana monitoring
- k3sup integration for easy node management
- Hybrid cloud automation
