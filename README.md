<h1>Automated Cloud Hosted WordPress Site.
<h1><b>(NOT FINISHED YET)</b>
<h2>Overview</h2>
Provisioning of Two Digital Ocean Droplets as WordPress Website Platforms Using Ansible Automation Tool.
<h2>Youtube Demo</h2>
(pending)
<br />

<h2>Environments Used </h2>

- <b>Digital Ocean Droplets</b>
- <b>Ubuntu 20.04</b>

<h2>Project Instructions:</h2>

<p align="center">
Network Topology: <br/>
<img src="https://i.imgur.com/g3hXiIz.png" width="80%" alt="Network Topology"/>
<br />

<section>
<h4>CONFIGURING THE UBUNTU SERVER</h4>

Create Ubuntu Server: <br/>
Using VMware Fusion, create a new virtual machine and select Ubuntu, make sure it is 20.04.6. <br/>
This will be our Linux server where Ansible will be installed.
<p align="center">
<br/>
<img src="https://i.imgur.com/BwVLot7.png" height="80%" width="80%" alt="Ubuntu Server Creation"/>
<br />
<br />

SSH into Ubuntu Server through Terminal: <br/>
Using the IP address of your Ubuntu Server, open a terminal window and use the SSH command to remotely connect to your server.
<p align="center">
<br/>
<img src="https://i.imgur.com/y0AUx4s.png" height="80%" width="80%" alt="SSH into Instances"/>
<img src="https://i.imgur.com/q2X6YPf.png" height="80%" width="80%" alt="SSH into Instances"/>

Configure Ubuntu Server: <br/>
Install Updates on Ubuntu Server. <br/>
  
```
sudo apt update
```
```
sudo apt upgrade
```
<p align="center">
<br/>
<img src="https://i.imgur.com/mCIGKhO.png" height="80%" width="80%" alt="Ubuntu Server Creation"/>
<br />
<br />

<h4>INSTALLING ANSIBLE AND DEPENDENCIES</h4>

Install Ansible. <br/>
  
```
sudo apt install ansible
```
```
ansible –version
```
<p align="center">
<img src="https://i.imgur.com/qxmL3UB.png" height="80%" width="80%" alt="db"/>

  
Install Python and Pip <br/>
  
```
sudo apt install python3
```
```
sudo apt install python3-pip
```
```
python3 –version
```
```
pip3 –version
```

Configure Ansible.cfg File <br/>
Navigate to the “ansible.cfg” file <br/>

```
cd /etc/ansible
```
```
sudo nano ansible.cfg
```

Remove the comment next to “inventory = /etc/ansible/hosts” under [defaults] <br/>
and change the path to your hosts file (mine was kept default). <br/>

Next, scroll down and uncomment the line “host_key_checking = False”. Then save the file.
  
<p align="center">
<br/>
<img src="https://i.imgur.com/NeuvdUE.png" height="80%" width="80%" alt="SSH into Instances"/>
<img src="https://i.imgur.com/T3aUK5G.png" height="80%" width="80%" alt="SSH into Instances"/>
<p align="center">

Configure Hosts File <br/>
Navigate to the path of your hosts file and add the following: <br/>
```
[digitalocean] or [localhost]
localhost ansible_connection=local
```
Replace localhost with the IP address of your server, then save the file.
<p align="center">
<br/>
<img src="https://i.imgur.com/vz4vIMj.png" height="80%" width="80%" alt="Hosts"/>

Create SSH key for Ansible: <br/>
Using the command below, create a new SSH key and then check to make sure it was created with the other command.
   ```
   ssh-keygen -t rsa
   ```
   ```
   cat .ssh/id_rsa.pub
   ``` 
<p align="center">
<img src="https://i.imgur.com/2fVJbAB.png" height="80%" width="80%" alt="sshkey"/>

<h4>CREATING ANSIBLE PLAYBOOK</h4>

Create New Ansible Playbook: <br/>
Inside of the /etc/ansible directory, run the following command to create a new script in YAML.

```
sudo nano name.yml
```
<p align="center">
<img src="https://i.imgur.com/BxXcHM8.png" height="80%" width="80%" alt="nano"/>
<br/>

Attach Digital Ocean account to Ansible: <br/>
In Digital Ocean, go down to API and select Tokens, then select Generate new token. Copy the ID and save it (we will be adding it to our script).<br/>
<p align="center">
<img src="https://i.imgur.com/3Kfealz.png" height="80%" width="80%" alt="token"/><br/>

Building the Ansible Script:<br/>
Below is a copy of the entire script you will add to your playbook (change the “digital_ocean_key” to match yours.

```
- hosts: digitalocean
  become: yes
  vars:
    digital_ocean_token: dop_v1_70d68f23d5ae71b86c683a41e50ce946230a268b1155fffac9583693facab652
    droplet_size: s-1vcpu-1gb
    droplet_region: nyc1
    droplet_image: CentOS-7-x64
    droplets:
      - RotermundG-202306-WP1
      - RotermundG-202306-WP2

  tasks:
#-------Check-SSH-Key-------
    - name: Check SSH Key
      user:
        name: '{{ansible_user_id}}'
        generate_ssh_key: yes
        ssh_key_file: .ssh/id_rsa

#-------Add-SSH-Key-To-DO-------      
    - name: Add SSH Key to Digital Ocean
      community.digitalocean.digital_ocean_sshkey:
        ssh_pub_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
        api_token: "{{ digital_ocean_token }}"
        state: present
      register: sshkey_result

#-------Create-Droplets-And-Assign-SSH-Key-------     
    - name: Create Droplet and Assign SSH Key
      community.digitalocean.digital_ocean_droplet:
        name: "{{ item }}"
        api_token: "{{ digital_ocean_token }}"
        size: "{{ droplet_size }}"
        region: "{{ droplet_region }}"
        image: "{{ droplet_image }}"
        unique_name: yes
        wait_timeout: 600
        ssh_keys: "{{ sshkey_result.data.ssh_key.id }}"
        state: present
      with_items: "{{ droplets }}"
      loop_control:
        pause: 3
      register: droplet_result

    - pause:
        seconds: 35
#-------LAMP-And-WordPress-Installation-------   
- hosts: digitalocean
  tasks:
    - name: Disable SeLinux
      become: yes
      selinux:
        state: disabled

    - name: Install LAMP
      yum: 
        name:
        - httpd
        - mariadb-server
        - php
        - nano
        - wget
        - rsync
        - firewalld
        - git-all

    - name: Install Epel Repo
      yum:
        name: epel-release
        state: latest
        update_cache: yes

    - name: Install Updates
      yum:
        name: '*'
        state: latest

    - name: Install Apache
      yum:
        name: httpd
        state: latest

    - name: Start Apache service
      become: yes
      service:
        name: httpd
        state: started
        enabled: yes
    - name: Install Firewalld
      yum:
        name: firewalld
        state: latest
    - name: Firewall HTTP Configuration
      become: yes
      firewalld:
        zone: public
        service: http
        permanent: yes
      state: enabled
    - name: Firewall SSH Configuration
      become: yes
      firewalld:
        zone: public
        service: ssh
        permanent: yes
        state: enabled
    - name: Reload Firewall
      become: yes
      service:
        name: firewalld
        state: reloaded

    - name: Install Python2
      yum:
        name: python2
        state: latest

    - name: Install Python3
      yum:
        name: python3
        state: latest

    - name: Install PIP
      pip:
        name: pip
        extra_args: --upgrade
        executable: pip3

    - name: Install Remi Repo
      yum:
        name: "https://rpms.remirepo.net/enterprise/remi-release-7.rpm"
        state: present

    - name: Continue PHP Install
      yum:
        enablerepo: "remi,remi-php80"
        name:
        - php
        - php-common
        - php-cli
        - php-gd
        - php-curl
        - php-mysqlnd
        - php-fpm
        - php-mysqli
        - php-json
        state: latest
        update_cache: yes

    - name: Install MariaDB
  - name: Install MariaDB
      yum:
        name: mariadb-server
        state: latest

    - name: Check MariaDB Status
      service:
        name: mariadb
        state: started    

    - name: Install MySQL Packages
      yum:
        name: ['python3-PyMySQL', 'python-PyMySQL']
        state: latest

    - name: Create MariaDB Database
      become: yes
      mysql_db:
        name: wordpress
        state: present

    - name: Create MariaDB Username & Password
      become: yes
      mysql_user:
        name: wordpress
        password: Jake$2005
        priv: "wordpress.*:ALL,GRANT"
        host: "localhost"
        state: present

    - name: Flush Privilege
      become: yes
      command: 'mysql -ne "{{ item }}"'
      with_items:
       - FLUSH PRIVILEGES

    - name: Restart MariaDB
      become: yes
      service:
        name: mariadb
        state: restarted

    - name: Download Wordpress
      become: yes
      get_url:
        url=http://wordpress.org/latest.tar.gz
        dest=/tmp/wordpress.latest.tar.gz
        validate_certs=no

    - name: Unzip Wordpress
      unarchive:
    src=/tmp/wordpress.latest.tar.gz
        dest=/var/www/
        copy=no

    - name: Give Apache Ownership
      file:
        path: "/var/www/wordpress"
        state: directory
        owner: "apache"
        group: "apache"
        mode: '0755'

    - name: Update Default Apache Site
      become: yes
      lineinfile:
        dest=/etc/httpd/conf/httpd.conf
        regexp="(.)+DocumentRoot /var/www/html"
        line="DocumentRoot /var/www/wordpress"

    - name: Restart Apache
      become: yes
      service:
        name: httpd
        state: restarted
        enabled: yes
```
<h4>RUNNING THE PLAYBOOK</h4>

Ansible Playbook Execution:<br/>
Now that the script has been created we can run it, make sure you are in the same directory of your playbook, using the following command run your playbook.<br/>

```
sudo ansible-playbook name.yml
```
</section>
<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
