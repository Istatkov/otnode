# Node Monitoring and Maintenance

## 1. **Monitor your node for free with NetData Cloud**

* Create an account on [NetData Cloud](http://netdata.cloud/)
* Install the application on your server

```text
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

Confirm “Y” and press Enter twice when asked during the installation.

Then you need to claim the node, through the claiming script. In Netdata Cloud Dashboard, and click on Claim Nodes. On the popup on the right, copy and paste and run the command displayed across all of your nodes.

![](../.gitbook/assets/image%20%285%29.png)

You will see a confirmation in the node that it was successfully claimed:

![](../.gitbook/assets/image%20%289%29.png)

On your registered e-mail you will start receiving notifications regarding anything related to your node – low RAM, low space, node is unreachable, etc. You can also set custom commands, and a I guide on those will be included below at a later time.

If your node is not discovered on the dashboard, you need to restart the process:

```text
sudo systemctl restart netdata
```

## **2. Get info on the locked and available tokens**

```text
docker exec otnode curl -s 'http://127.0.0.1:8900/api/latest/balance?humanReadable=true'
```

Here are few commands to use to maintain your node.

## **3. Perform manual job payouts**

Go to the My nodes section on [OTHUB](https://othub.origin-trail.network/nodes/mynodes) and open the node you want to make payouts for. In the jobs section find the jobs which are Completed and not paid out yet:

![](../.gitbook/assets/image%20%2825%29.png)

Option 1: Click on Payout and follow the instructions

Option 2: Payout from the node terminal

Run the below command by replacing the XXXXX with the job offer ID, listed on the same row, which you obtained from [OTHUB](https://othub.origin-trail.network/nodes/mynodes).

```text
docker exec otnode curl -s -X GET http://127.0.0.1:8900/api/latest/payout?offer_id=XXXXX
```

**Important**: This command will use ETH from the Operational wallet. Due to the high GWEI, make sure you have set a “max\_allowed\_gas\_price”: setting in your .origintrail\_noderc configuration file, to limit the expenses.

Alternatively you can obtain a list of offers you can payout \(however this includes all offers including those that just started, which could payout a very minimal amount\) \(replace XXXX below with your node ID\):

```text
curl -X GET "https://othub-api.origin-trail.network/api/nodes/DataHolders/XXXXXXXXXXXX" -H "accept: text/plain" | grep -o -P '(?<=OfferId":").*?(?=","FinalizedTimestamp")'
```

## **4. Update the applications on the server**

```text
apt update && apt upgrade -y
```

## **5. Check logs for the past 48 hours**

```text
docker logs --since 48h otnode | more
```

## **6. Restart the node**

```text
docker restart otnode
```

## **7. Remove swap space**

```text
cat /proc/swaps
```

```text
sudo swapoff /swapfile
```

```text
sudo rm /swapfile
```

```text
nano /etc/fstab
```

Then remove the following from the file and save it: **/swapfile swap swap defaults 0 0**

## 8**. Alternative way to configure the node is through Houston tool, which you can find here \(please note this version requires SSL\):**

[https://github.com/OriginTrail/houston-v4](https://github.com/OriginTrail/houston-v4)

Alternatively you can use this version which has the SSL version disabled:

[https://github.com/Guinnessstache/houston-v4](https://github.com/Guinnessstache/houston-v4)

## **9. I have disabled auto payout, but for old jobs they keep trying to payout with error “info – Gas price too high, delaying call for 30 minutes” but it appears to be popping up every 5 seconds.**

Open the bash console inside your node with the following command:

```text
docker exec -it otnode bash 
```

Go to the data section of your node:

```text
cd /ot-node/data
```

Open the SQLite shell of your database

```text
sqlite3 system.db
```

Add a 30 minute period \(in milliseconds\) for all payout commands

```text
update commands set period=1800000 where name="dhPayOutCommand";
```

Exit the SQLite shell and the container by pressing Control+D twice.

Restart the node

```text
docker restart otnode
```

Due to frequent node access to the database, some commands listed above might fail, but you can just run the command again. 

