# Incremental Backup

For those who are running multiple nodes, you can automate the process of installing and managing your nodes through utilizing ansible. This section was brought to you by Calvin \(@calr0x\) and you can find the latest version of his work and tools on his github here: [https://github.com/calr0x](https://github.com/calr0x). Should you have any questions, feel free to join the dedicated group for advanced node runners at [https://t.me/otnodes](https://t.me/otnodes)

First you need a “Control” Server to manage your nodes. You can use your own Windows box, one of your nodes or a Storage VPS you use to store your backups on.

**Own Windows server**

If you use Windows 10, you can download a Virtual Machine software from Oracle

[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

![](https://otnode.com/wp-content/uploads/2021/06/image.png)

Once you download it, you need the Ubuntu 20.04 \(or whatever is latest\) distribution at: [https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)

Setup a New Virtual Machine with 2 GB Ram and 10 GB HDD space

![](https://otnode.com/wp-content/uploads/2021/06/image-1.png)

New VM Setup

![](https://otnode.com/wp-content/uploads/2021/06/image-5.png)![](https://otnode.com/wp-content/uploads/2021/06/image-3.png)

Virtual drive selection

![](https://otnode.com/wp-content/uploads/2021/06/image-6.png)![](https://otnode.com/wp-content/uploads/2021/06/image-7.png)

Once the image is setup, right click, settings, general, advanced, select bidirectional for shared clipboard and Drag n drop

![](https://otnode.com/wp-content/uploads/2021/06/image-8.png)

Then go to display tab, screen, max out video memory to 128mb

![](https://otnode.com/wp-content/uploads/2021/06/image-9.png)

Then Settings -&gt; Storage -&gt; Controller: IDE -&gt; Empty and load the Ubuntu ISO container you downloaded through clicking on the blue disk and selecting “Choose a disk file” and click ok.![](https://otnode.com/wp-content/uploads/2021/06/image-12.png)

Then click the green arrow Start, and click start on the popup.

![](https://otnode.com/wp-content/uploads/2021/06/image-13.png)

Then choose Install Ubuntu:

![](https://otnode.com/wp-content/uploads/2021/06/image-14.png)

Then choose between normal or minimal installation

![](https://otnode.com/wp-content/uploads/2021/06/image-15.png)

Then click Install Now

![](https://otnode.com/wp-content/uploads/2021/06/image-16.png)

Then fill details about location and login and finish the installation.

Once the desktop loads, right click on it and choose Open in Terminal![](https://otnode.com/wp-content/uploads/2021/06/image-17.png)

Then follow the commands as you would do on a VPS. If your clipboard is not working, you need to run these three commands first, type them inside the terminal window

```text
sudo -iCopy
```

enter your password to confirm, so you can switch to root

```text
apt-get updateCopy
```

```text
apt-get install virtualbox-guest-x11Copy
```

```text
VBoxClient --clipboardCopy
```

**Install Ansible**

```text
sudo apt update -y && sudo apt upgrade -yCopy
```

```text
apt-get install git -yCopy
```

Then you need to install Ansible and the relevant tools we will use. These are developed by Calvin \(@calr0x\) and you can find the latest version of his work and tools on his github here: [https://github.com/calr0x](https://github.com/calr0x)

```text
apt update && apt install ansibleCopy
```

```text
git clone https://github.com/calr0x/OT-Ansible-Files-and-Playbooks.gitCopy
```

If you use SSH to authenticate with your nodes, you need to upload the SSH private key and public key to the control machine \(either virtual or VPS\). You do that by the following:

From the root directory:

```text
cd .ssh/Copy
```

```text
nano id_rsaCopy
```

Paste the private key of your SSH, then CTRL + S \(to save\) and CTRL + X \(to exit nano\)

```text
chmod 700 id_rsaCopy
```

```text
nano id_rsa.pubCopy
```

Paste the SSH Public key inside, then CTRL + S \(to save\) and CTRL + X \(to exit nano\)

```text
cd ..Copy
```

```text
nano /etc/ansible/ansible.cfgCopy
```

remove the \# in front of host\_key\_checking = False so it looks like the below, then CTRL + S \(to save\) and CTRL + X \(to exit nano\)![](https://otnode.com/wp-content/uploads/2021/06/image-18.png)

**Setting up the configuration file with all the nodes**

```text
cd OT-Ansible-Files-and-PlaybooksCopy
```

```text
nano config-otnodes-and-cosmic.ymlCopy
```

File for managing exiting nodes:

Important: Spaces are very important when setting up the configuration file. otnodes: has no spaces in front, hosts: has 2 spaces, the IP below it 4 spaces, the node\_name: has 6 spaces. Add as many nodes as you want to manage.

```text
otnodes:
  hosts:
    xxx.xxx.xxx.xx:
      node_name: 'XXX'

    xxx.xxx.xxx.xx:
      node_name: 'XXX'
Copy
```

OT Node Watch

This script will monitor if your nodes are bidding every XX minutes \(set at 15 by default\) and will notify you if a node is not bidding. This would mean the node might have run out of space, or has hanged, and would need a restart

```text
git clone https://github.com/calr0x/OT-NodeWatch.gitCopy
```

Generate Telegram Bot token

Send a message to

```text
@BotfatherCopy
```

with the message

```text
/newbotCopy
```

and choose a name for your bot and you will obtain the token. Then you can choose to either get messages as pms from your bot, or you can add it to a group and get the messages in the group. If you choose PMs, message the following bot:

```text
@myidbotCopy
```

and send him the following message and save the number you receive, which is your CHAT\_ID

```text
/getidCopy
```

If you want to receive the messages in a group, add the above @myidbot in the group and type in the group:

```text
/getgroupidCopy
```

Once you have the bot token and chat id noted, from root enter:

```text
cd OT-NodeWatch/Copy
```

```text
nano config.shCopy
```

Enter these 2 values there and CTRL + S \(to save\) and CTRL + X \(to exit nano\)

```text
cd ansibleCopy
```

```text
nano install-monitors.ymlCopy
```

![](https://otnode.com/wp-content/uploads/2021/06/image-19.png)

Change the bidcheck\_minute variable to your preference. If you want it to check every hour, set to bidcheck\_minute: ‘0’, so you don’t get spam when there is downtime of jobs.

Running playbooks

Once your configuration is ready, you run the following commands form the root directory if you have an SSH with password:

```text
ssh-agent bashCopy
```

```text
ssh-addCopy
```

Type your password for your SSH

Then from root start:

```text
ansible-playbook /root/OT-NodeWatch/ansible/install-monitors.ymlCopy
```

Backups with SmoothBrain ansible playbook

```text
git clone https://github.com/calr0x/OT-Smoothbrain-Backup.gitCopy
```

Work in progress

