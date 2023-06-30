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

<h2>Project walk-through:</h2>

<p align="center">
<! --- Network Topology: <br/>
<! --- <img src="https://i.imgur.com/IoEpRBS.png" height="80%" width="80%" alt="Network Topology"/>
<! --- <br />
<! --- <br />

1.  Create Ubuntu Server:
Using VMware Fusion, create a new virtual machine and select Ubuntu, make sure it is 20.04.6. <br/>
This will be our Linux server where Ansible will be installed.
<p align="center">
<br/>
<img src="https://i.imgur.com/BwVLot7.png" height="80%" width="80%" alt="Ubuntu Server Creation"/>
<br />
<br />

2.  SSH into Ubuntu Server through Terminal: <br/>
Using the IP address of your Ubuntu Server, open a terminal window and use the SSH command to remotely connect to your server.
<p align="center">
<br/>
<img src="https://i.imgur.com/y0AUx4s.png" height="80%" width="80%" alt="SSH into Instances"/>
<img src="https://i.imgur.com/q2X6YPf.png" height="80%" width="80%" alt="SSH into Instances"/>
<br />
Configure Ubuntu Server:
  
3.	Install Updates on Ubuntu Server.
  
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
  
4.	Install Ansible.
  
    ```
    sudo apt install ansible
    ```
    ```
    ansible –version
    ```
<p align="center">
<img src="https://i.imgur.com/qxmL3UB.png" height="80%" width="80%" alt="db"/>

  
 5.	Install Python and Pip
  
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

6.  Configure Ansible.cfg File
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

7. Configure Hosts File
Navigate to the path of your hosts file and add the following: <br/>
```
[digitalocean] or [localhost]
localhost ansible_connection=local
```
Replace localhost with the IP address of your server, then save the file.
<p align="center">
<br/>
<img src="https://i.imgur.com/vz4vIMj.png" height="80%" width="80%" alt="Hosts"/>
<p align="center">
<br />

8. Create SSH-Keygen
   ```
   ssh-keygen -t rsa -b 4096 -f /path/to/your_key_name
   ```
You will have to specify the path to your .ssh file and then name the key what you want. <br/>
Once done, you can navigate to the .ssh directory and “ls” to list all files and verify the keys were created.  

<p align="center">
<br/>
<img src="https://i.imgur.com/Di80hLJ.png" height="80%" width="80%" alt="Hosts"/>
<img src="https://i.imgur.com/Fjk2rGF.png" height="80%" width="80%" alt="Hosts"/>
<p align="center">
<br />
  
<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
