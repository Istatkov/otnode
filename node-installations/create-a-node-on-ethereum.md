# Create a node on Ethereum

Here is a video guide I made for v4, it is very similar for ethereum for V5 releasing in 23 March 2021, though I would hold off on setting up nodes on Ethereum until the gas issue is not resolved by the Ethereum developers. You can setup nodes on xDAI instead and on SFC in Q2.

**Create operational wallet**

Create a new wallet using Mycrypto or Metamask and export the private key, which we will use for the operational wallet for the node. Deposit 3000 TRAC and 0.15 eth. The 3000 TRAC is locked along with the creation of the node identity, which can be retrieved should you decide to stop hosting a node. The ETH is for gas fees during installation and litigation responses.

**Register on Infura**![This image has an empty alt attribute; its file name is image-2.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2F9CtKU4lwXPx5pO3cMdJv%2Ffile.svg?alt=media)

Go to [Infura.io](https://infura.io) and create an account. Create a project, copy the HTTPS link under Endpoints – Mainnet. It should look like: https://mainnet.infura.io/v3/xxxxxxxxxxxxxxxxxxxxxx where the red letters are your specific link![This image has an empty alt attribute; its file name is image-8.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2F5D56OrjZW3Uby4CFDOJ7%2Ffile.svg?alt=media)

**Configure the server**

Download [Termius](https://www.termius.com) (or any other terminal client like [Kitty](https://www.fosshub.com/KiTTY.html)) and configure it with the details you received from the VPS hosting (IP, username, password). Click on Hosts, Select New host, Choose a Label for the node and add the IP address from the confirmation e-mail from Digital Ocean or Hetzner that the node is created, choose root as username and input the password, and click on Save on the right top corner.![This image has an empty alt attribute; its file name is image-3-979x1024.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FFkcsjcsR4yvWTNRtORoh%2Ffile.svg?alt=media)

Once you login follow the configuration logic below. Click on the COPY button after each command and **right click** into the terminal window to paste it. Then confirm with Enter

**1. Update the server programs:**

```
apt update && apt upgrade -y
```

![This image has an empty alt attribute; its file name is image-4.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FPOA5CHvvFTTunIpTvb7e%2Ffile.svg?alt=media)

**2. Reboot server with below command, close the window and login again**

```
reboot
```

**3. Install docker** (skip this step if you selected the Digital Ocean Server with Docker installation).

The official installation commands for docker can be found here, should the ones in this section become outdated: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

```
apt-get install apt-transport-https ca-certificates curl software-properties-common
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs)  stable"
```

```
apt-get update
```

```
apt-get install docker-ce
```

**4. Setup the firewall**

```
ufw allow 8900
```

```
ufw allow 5278
```

```
ufw allow 3000
```

**4a. Check if firewall is setup**

```
ufw allow 22/tcp
```

```
ufw show added
```

**4b. Activate and double check the firewall is properly configured:**

```
ufw enable
```

```
ufw status
```

![This image has an empty alt attribute; its file name is image-5.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2Fq6w6Vy3glHM7o6FiNo89%2Ffile.svg?alt=media)

**5. Setup the configuration file**

Step A: Check [ethgasstation.info](https://ethgasstation.info) to see what is the current GWEI![This image has an empty alt attribute; its file name is image-1.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FeZPHOdIID0HC7SuTqJP2%2Ffile.svg?alt=media)

Your node will create the identity with the FAST value. With 91 GWEI it will use around 0.1 eth to do so. Make sure the max\_allowed\_gas\_price setting below is above the current gwei to allow for the transaction to be processed.

```
nano /root/.origintrail_noderc
```

Paste the below in the text editing tool and update the red letters accordingly:

```
{
   "blockchain": {
     "implementations": [
       {
         "blockchain_title": "Ethereum",
         "network_id": "ethr:mainnet",
         "identity_filepath": "erc725_identity.json",
         "dh_price_factor" : "0.1",
         "node_wallet": "xxxxxxxx",
         "node_private_key": "xxxxxxxx",
         "management_wallet": "xxxxxxxx"
         "rpc_server_url": "xxxxxxxxx"
       }
     ]
   },
   "network": {
     "hostname": "xxx.xxx.xxx.xxx"
     },
   "initial_deposit_amount": "5000000000000000000000",
   "dh_max_holding_time_in_minutes": 5256000,
   "dh_maximum_dataset_filesize_in_mb": 10
}
```

save the file by **Ctrl-O** and **Enter** and exit by **Ctrl-X**

**Explanation:**

**node\_wallet** – Operational wallet public address

**node\_private\_key** – Operational wallet private key. As you will store just 0.1 – 0.15 ETH there for gas fees, even if server gets compromised, the hacker would have access only to this small amount

**Note**: If your private key start with **0x**, remove these two characters when adding it to the configuration file.

**management\_wallet** – this is your management wallet public address – ideally this should be a wallet on your cold storage ([Ledger Nano S](https://shop.ledger.com/pages/ledger-nano-x?r=c912d8040032), etc.)

**dh\_price\_factor** – this is the setting of your lambda value – should be less than 1 if you want to accept most of all current jobs. The lower the value, the less paid jobs you are willing to accept.

**Important**:

**“max\_allowed\_gas\_price”** – this is the limit of wei the node will set to create the identity, payout jobs, etc. If the value is lower than the current average value of ethgasstation, it will be postponed and tried later. Be careful with that setting, and adjust accordingly to your preference how much you are willing to pay for those transactions. 50000000000 is equal to 50 gwei, so for the node creation you might want to increase to set accordingly.

**rpc\_server\_url** – this is your infura https address which you obtained during the registration there.

**Hostname** – this is your server IP, which you can find in the notification e-mail when you setup the VPS

**remoteWhitelist**: leave at 127.0.0.1

**dh\_max\_holding\_time\_in\_minutes** – the maximum length of jobs you are willing to accept in minutes (for example 550 000 is to accept one year jobs).

**disableAutoPayouts** – disable automatic payouts so you can process those manually when the ETH network gwei is low.

**6. Install JQ to validate whether the configuration file doesn’t contain any errors**

```
apt-get install jq
```

```
jq "." /root/.origintrail_noderc
```

Check the brackets, double quotes and commas. Everything except your data has to be exactly like the example above.

**7. Initiate the node installation**

Before running the command, check what is the GWEI on [ethgasstation.info](https://ethgasstation.info), as this command uses lots of GAS. Ideally you would like to do it when it is around 30 GWEI (late GMT evening time), and adjust the gas setting accordingly.

**Important**: Once you run this command, the 3000 TRAC will be deposited to the contract and your first task is to copy and back your ERC725 identity and node identity mentioned on Step 8 below, so you can have control over this deposited amount. Should you get an error at this stage, or if the gas setting you used is too low, do not delete the container or destroy the VPS. Instead join the discord channel or the Official Telegram group to ask for assistance

```
sudo docker run -i --log-driver json-file --log-opt max-size=1g --name=otnode -p 8900:8900 -p 5278:5278 -p 3000:3000 -v ~/.origintrail_noderc:/ot-node/.origintrail_noderc origintrail/ot-node:release_mainnet
```

Once completed and node has joined the network, exit the log with CTRL +C

**Important**: If you get a warning that the gwei is too low and it will retry in 30 minutes, and you want to increase the gwei, scroll to the bottom of the guide, End Note 1, what to do.

**8. Create identities into files on the root directory**

ERC725 identity

```
docker cp otnode:/ot-node/data/erc725_identity.json ~/erc725_identity.json
```

**Important:** If you interrupted the ERC725 creation and the file was not created, but tokens were sent to the contract, and you get this error: “error: no such container:path: otnode:/ot-node/data/erc725\_identity.json”, scroll to the End Note 2 how to extract the ERC725 identity and manually create it.

Node ID

```
docker cp otnode:/ot-node/data/identity.json ~/identity.json
```

**9. Read erc725 and node id and copy/paste them in secure document**

```
more erc725_identity.json
```

```
more identity.json
```

Or Login to the server using WINSCP or any other FTP application, go to the root folder and download these two files on your local server and archive them with a password.

**10. Deposit TRAC from your management wallet to the node**

Deposit the amount of TRAC which will be available for jobs through the node profile management page at this URL: [https://node-profile.origintrail.io/](https://node-profile.origintrail.io)

The current recommended amount is 5000 – 7000 TRAC, and they have to be present on your management wallet, which should be on a cold storage device like [Ledger Nano S](https://shop.ledger.com/pages/ledger-nano-x?r=c912d8040032). If you have not secured your crypto holdings on cold storage, please do so now as this is the best way to keep your funds secure.

Login with your **ERC725 Identit**y which you extracted above and the **operational wallet public address** and follow the instructions on the page. You need metamask to initiate the deposit.

Then on the top left section “Deposit TRAC to Your Node”, enter the amount of TRAC present on your management wallet, to be deposited to the contract for your node, and a set of 2 transactions have to be processed. Metamask popup will show up for you to confirm you want to process the transaction. Once the first transaction is processed, a second popup will show for the second transaction.

**Note:** Users reported that Metamask calculates the transaction with the maximum possible gas fee of 8.9 mil gas, and as such would display the transaction will cost more than $50 USD. Should this happen, click on Edit and adjust the gas limit to 200 000 only.![This image has an empty alt attribute; its file name is image-9.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FpvauaPoqtA5i7NzpBZKM%2Ffile.svg?alt=media)

Once the deposit is completed, restart your node.

**11. Restart the node**

```
docker stop otnode
```

```
docker start otnode
```

Update the docker container to auto restart the container (new command)

```
docker update --restart=always otnode
```

**12. Follow the log to monitor the operation of the node**

```
docker logs -f otnode
```

**13. Monitor your node**

You can add your node to the OT Hub and monitor how many jobs has the node won, initiate manual payouts and quickly check if your node is online. The OT Hub can be found on the link below:

&#x20;https://othub.origin-trail.network/dashboard

Also in the **Node Maintenance** section (menu on the right) you can find how to setup notifications on your mobile.

**14. Add swap space**

Swap space is dedicated space on your hard drive, which is used as RAM should the hardware RAM is fully utilized. This usually slows down the server as the hard drive is slower than the RAM, however the swap would ensure the node will continue operation. I highly recommend to enable 1 GB Swap space on your server.

* check if there is swap space already enabled (SWAP line should show 0)

```
free -h
```

* Add 1 GB Swap space

```
fallocate -l 1G /swapfile
```

* enable the swap file

```
chmod 600 /swapfile
```

* mark the space as swap file

```
mkswap /swapfile
```

* enable for utilisation

```
swapon /swapfile
```

* confirm it is setup (SWAP line should show 1)

```
free -h
```

![This image has an empty alt attribute; its file name is image-4.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FUEW3wYniQ4frMJLYWfyP%2Ffile.svg?alt=media)

* make it permanent

```
cp /etc/fstab /etc/fstab.bak
```

```
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**15. Install Fail2ban**

This tool allows you to ban any user who failed to login successfully to the server after three attempts. This ensures your server cannot be bruteforced by automated bots, running millions of combinations to try to guess your password.

```
apt install -y fail2ban
```

```
systemctl start fail2ban
```

```
systemctl enable fail2ban
```

Configure the application

```
nano /etc/fail2ban/jail.local
```

Paste the following content in the file editor

```
[sshd] 
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
```

Save with CTRL + O and Enter and exit with Ctrl + X. Then restart the process with

```
systemctl restart fail2ban
```

**16. Setup e-mail alerting for Digital Ocean**

If you are using Digital Ocean, you can setup an alert to notify you by e-mail when your CPU load is below a certain threshold (i.e. 3%), which would most probably mean the node is offline.

In the manage section in the user area, select Monitoring and click on Create Alert Policy.![This image has an empty alt attribute; its file name is image-1-1024x284.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FA1P19KUGM4d1afof1vfY%2Ffile.svg?alt=media)

Configure the alert as below![This image has an empty alt attribute; its file name is image-3-1024x571.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2F6PZ10UlkyYMk48GcGpEV%2Ffile.svg?alt=media)

## **End Notes (regular installation stops here):**

End Note **1. Increase the gwei limit when installing the node**

Important: Before doing this, make sure that:

* the 3000 TRAC tokens are not submitted to the contract (check the operational wallet on etherscan.io)
* check the log to confirm the node is between the 30 minutes waiting period (docker logs otnode -f)

Hit Ctrl + C and stop the node with the command below

```
docker stop otnode
```

```
nano /root/.origintrail_noderc
```

Update the “max\_allowed\_gas\_price”: “30000000000” setting to whatever you are comfortable paying. i.e. replace 3 with 5 to increase from 30 gwei to 50 gwei. If you this step with 90 gwei, it will take \~0.1 eth, so be warned.

save the file by **Ctrl-O** and **Enter** and exit by **Ctrl-X**

```
docker start otnode
```

```
docker logs otnode -f
```

End Note **2. Obtain and create manually the ERC725 identity, if you got an error at step 8.**

Go to [Etherscan.io](https://etherscan.io) and search for your operational wallet public address![This image has an empty alt attribute; its file name is image-6-1024x402.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FtWsunYmfEdxqiHwnG9Ta%2Ffile.svg?alt=media)

Click on the ERC20 Token Txns section (1) and locate the outgoing transaction of 3000 TRAC and click on the respective Txn Hash (2) on the left.![This image has an empty alt attribute; its file name is image-7-1024x374.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FN5BI6Sdx8V10T0NF7aVK%2Ffile.svg?alt=media)

Then on the Transaction Hash screen click on the Internal Txns (1) and you will see the ERC725 identity there (2). Click on it.![This image has an empty alt attribute; its file name is image-8-1024x325.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FLSkaNbIUCNEVbAswM5Cr%2Ffile.svg?alt=media)

Click to copy the ERC725 identity (1), which is basically the Contract number on the left of the button.![This image has an empty alt attribute; its file name is image-10-1024x294.png](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-McTpPEgHESZRgkZ2xAE%2Fuploads%2FsWsxLeUJMOipcS8zAOaC%2Ffile.svg?alt=media)

Then through the terminal application (i.e. Termius) login to the server and enter the following commands while updating the XXXXXX with the identity you copied on the step above

```
echo "{\"identity\":\"XXXXXXXXXXXXXXXXXXX\"}" > erc725_identity.json  
```

Insert the identity file into the OT docker container with the following command:&#x20;

```
docker cp erc725_identity.json otnode:/ot-node/data/ 
```

Restart the node so the new identity can kick in.

```
docker stop otnode
```

```
docker start otnode
```

```
docker logs otnode -f
```
