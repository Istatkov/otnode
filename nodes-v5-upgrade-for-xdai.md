# Nodes V5 upgrade for xDAI

1. Confirm your node has the latest version

```text
docker logs otnode | grep "Version check" | tail -1
```

If your node doesn’t have 4.1.17, restart your docker container and wait for it to update.

![](.gitbook/assets/image%20%2819%29.png)

Also check if you have enough space on your node.

```text
df -h
```

![](.gitbook/assets/image%20%2812%29.png)

If you are running out of space, check how to [cleanup your server](maintenance/node-space-management.md) and if you are still above 70%, either upgrade your server or add additional volumes.

2. Backup your node

Below is a pic of the recent fire in one of OVH datacenters, which wiped out 3+ million websites which had no backup. Please backup your node now! Instructions how to do that here: [https://otnode.com/node-backup/](maintenance/node-backup.md)

![](.gitbook/assets/image%20%284%29.png)

3. Obtain xDAI and xTRAC

First you need xDAI, which is the gas fee currency on xDAI. Given that the gas fees there are very low, 1-2 xDAI will last you a long time. There are multiple ways to obtain it:

* You can obtain DAI from [Uniswap](https://info.uniswap.org/token/0x6b175474e89094c44da98b954eedeac495271d0f) or [Curve](https://curve.fi/) . Then you convert it through the bridge [https://bridge.xdaichain.com/](https://bridge.xdaichain.com/) – here is a video tutorial how to do it: [https://www.youtube.com/watch?v=oKdh2cOOqUs](https://www.youtube.com/watch?v=oKdh2cOOqUs) This operation would cost one Ethereum smart contract transaction.
* Buy xDAI directly from [Bitmax](https://bitmax.io/en/basic/cashtrade-spottrading/usdt/xdai). If you have USDT on Binance, send the USDT through TRX network to Bitmax \(so it costs $1 for the transfer\).
* Use the faucet to get 0.01 xdai -&gt; [https://blockscout.com/xdai/mainnet/faucet](https://blockscout.com/xdai/mainnet/faucet)
* If the faucet is not working, join the OriginTrail Community telegram and as for a cent of xdai -&gt; [https://t.me/OriginTrailCommunity](https://t.me/OriginTrailCommunity)

Second you need xTRAC. If you participated in the SFC boarding, you will obtain back 5% of your TRAC contribution as xTRAC bounty. To claim those, please visit the staking website once announced by the team.

You can also convert TRAC tokens to xTRAC using the Omni Bridge. Go to [https://omni.xdaichain.com/](https://omni.xdaichain.com/) Instructions how to use the bridge can be found here: [https://docs.tokenbridge.net/eth-xdai-amb-bridge/multi-token-extension/ui-to-transfer-tokens/transfer-erc20](https://docs.tokenbridge.net/eth-xdai-amb-bridge/multi-token-extension/ui-to-transfer-tokens/transfer-erc20)

Once you have sorted out your xDAI and xTRAC, you need to add the Custom RPC for xDAI, so you can switch your Metamask to the xDAI network. Easiest way to do it is to go to: [https://chainlist.org/](https://chainlist.org/) and click add me to metamask button on xDAI section.

Also the custom token contract to see xTRAC on your Metamask is:

```text
0xEddd81E0792E764501AaE206EB432399a0268DB5
```

![](.gitbook/assets/image%20%2827%29.png)

4. Migration script

Extract the migration script:

```text
curl -O https://raw.githubusercontent.com/OriginTrail/ot-node/feature/update-migrate-script/scripts/migrate_to_v5.sh
```

Set it as executable:

```text
chmod +x migrate_to_v5.sh
```

```text
./migrate_to_v5.sh
```

Your node will restart, and you need to open the log file and monitor if the upgrade has been successful and the node has connected to the network.

```text
docker logs otnode --tail 1000 -f
```

5. Setup the identity on the VPS

```text
nano /root/.origintrail_noderc
```

Then update the config file on your node with the following data :

```text
{
   "blockchain": {
     "implementations": [
       {
         "blockchain_title": "Ethereum",
         "network_id": "ethr:mainnet",
         "identity_filepath": "erc725_identity.json",
         "dh_price_factor" : "1",
         "node_wallet": "xxxxxxxx",
         "node_private_key": "xxxxxxxx",
         "management_wallet": "xxxxxxxx",
         "rpc_server_url": "xxxxxxxxxxx"
       },
       {
         "blockchain_title": "xDai",
         "network_id": "xdai:mainnet",
         "identity_filepath": "xdai_erc725_identity.json",
         "dh_price_factor" : "1",
         "node_wallet": "xxxxxxxx",
         "node_private_key": "xxxxxxxx",
         "management_wallet": "xxxxxxxx"
       } 
     ]
   },
   "network": {
     "hostname": "xxx.xxx.xxx.xxx"
     },
   "initial_deposit_amount": "5000000000000000000000",
   "dh_max_holding_time_in_minutes": 530000,
   "disableAutoPayouts": true,
   "dh_maximum_dataset_filesize_in_mb": 10
}
```

**Initial\_deposit\_amount** – amount of xTRAC you want to deposit on your node during the installation \(so you skip step 10\). \(5000000000000000000000 is equal to 5000 xTRAC\). Make sure the amount is deposited before you initiate next step.

Restart your node

```text
docker restart otnode
```

**6. Create identities into files on the root directory**

ERC725 identity

```text
docker cp otnode:/ot-node/data/xdai_erc725_identity.json ~/xdai_erc725_identity.json
```

Node ID

```text
docker cp otnode:/ot-node/data/identity.json ~/identity.json
```

**7. Read erc725 and node id and copy/paste them in secure document**

```text
more xdai_erc725_identity.json
```

```text
more identity.json
```

Important: If you get an error where it says you have not enough tokens to create identity and it will retry, but your identity has already been created follow the steps below**.**

Go to [OTHUB](https://v5.othub.info/) and look the transaction which your operational wallet completed \(example\)![](data:image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201024%20601'%3E%3C/svg%3E)

![](.gitbook/assets/image%20%2818%29.png)

![](.gitbook/assets/image%20%2830%29.png)

Copy the xDAI identity to a notepad, and then through the terminal application \(i.e. Termius\) login to the server and enter the following commands while updating the XXXXXX with the identity you copied on the step above

```text
echo "{\"identity\":\"XXXXXXXXXXXXXXXXXXX\"}" > xdai_erc725_identity.json  
```

Insert the identity file into the OT docker container with the following command: 

```text
docker cp xdai_erc725_identity.json otnode:/ot-node/data/ 
```

Restart the node so the new identity can kick in.

```text
docker restart otnode
```

