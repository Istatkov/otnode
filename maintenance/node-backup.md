# Node Backup

## Backing up your node

You can backup your node on AWS, where all relevant data will be secured in case there is an issue with your VPS, so you can restore it. You can choose backup service from your VPS provider as well, though these cost usually much more than AWS. In order to setup the backup of your node, please follow these steps:

### 1. Setup your bucket

Go to [https://s3.console.aws.amazon.com/s3/home?region=eu-west-1](https://s3.console.aws.amazon.com/s3/home?region=eu-west-1) and create a bucket:

![](../.gitbook/assets/image%20%2814%29.png)

Choose name of the bucket \(otnode for example\) and leave the other options by default. Once you create the bucket

### 2. Generate your Access Key and Secret key – click on your user on the top right corner and select My Security Credentials:

![](../.gitbook/assets/image%20%2834%29.png)

Click on “Access keys \(access key ID and secret access key\)” and click on “Create new access key” and download it. On the screen and in the file you will see your AWSAccessKeyId and your AWSSecretKey.

![](../.gitbook/assets/image%20%2815%29.png)

### 3. Create the backup script inside the terminal of your node:

```text
nano aws-upload.sh
```

Then paste inside the below command while updating the Red text with your AWSAccessKeyId, AWSSecretKey and Bucket name

```text
docker exec otnode node scripts/backup-upload-aws.js --config=/ot-node/.origintrail_noderc --configDir=/ot-node/data --backupDirectory=/ot-node/backup --AWSAccessKeyId=YOUR_AWS_ACCESS_KEY_ID --AWSSecretAccessKey=YOUR_AWS_SECRET_ACCESS_KEY --AWSBucketName=YOUR_AWS_BUCKET_NAME
```

Save with Ctrl + O and Enter, and exit with Ctrl + X

Then change the script permissions to be executable with:

```text
chmod +x aws-upload.sh
```

Then you can run the script with:

```text
./aws-upload.sh
```

The result should look like this:

![](../.gitbook/assets/image%20%2833%29.png)

### 4. Setup a cron job to run weekly:

```text
crontab -e
```

You will most probably see this output:

![](../.gitbook/assets/image%20%2817%29.png)

Enter 1 and click Enter

Add at the bottom another line with the following text for backup to be performed every day at noon \(you can setup a custom time, i.e. weekly using this [Crontab Guru\)](https://crontab.guru/):

Daily at noon:

```text
0 12 * * * ~/aws-upload.sh >> /root/aws-backup.log
#
```

Weekly at Sunday midnight:

```text
0 0 * * 0 ~/aws-upload.sh >> /root/aws-backup.log
#
```

Save with Ctrl + O and Enter, and exit with Ctrl + X

![](../.gitbook/assets/image%20%288%29.png)

## **Restoring your node with AWS CLI**

Prepare a new VPS and run steps 1-4 from the main guide, then proceed below

### 1. Install AWS CLI

```text
docker run --rm -it amazon/aws-cli --version
```

### 2. Setup your configuration

```text
docker run --rm -ti -v ~/.aws:/root/.aws amazon/aws-cli configure
```

Enter your AWSAccessKeyId, AWSSecretKey, Region, and enter None for output

### 3. Test your configuration

```text
docker run --rm -ti -v ~/.aws:/root/.aws amazon/aws-cli s3 ls
```

You should see a list of all your buckets

### 4. Retrieve the backup files from AWS to your current directory

Replace **yourbucket** below with the bucket of your node \(and keep the dot at the end\) \(NOTE: This will download all backups from AWS if you have more than one in that folder. To save space, login to AWS and delete all old ones, and leave the most up-to-date one that you will use\).

```text
docker run --rm -it -v ~/.aws:/root/.aws -v $(pwd):/aws amazon/aws-cli s3 sync s3://yourbucket .
```

### 5. Rename the downloaded files to backup

The folder you downloaded is named with name+IP. We need to rename it to “Backup”. Easiest is to write mv then write the first letter of the folder and cycle with tab until you match the name.

mv downloadedfolder backup

### 6. Copy the config file

```text
cp /root/backup/.origintrail_noderc /root/.origintrail_noderc
```

```text
If the IP is different, update it accordingly with nano editor
```

nano /root/.origintrail\_noderc

### 7. Install the otnode container

If you are installing a v5 backup, use the below

```text
sudo docker create -i --log-driver json-file --log-opt max-size=1g --name=otnode -p 8900:8900 -p 5278:5278 -p 3000:3000 -v ~/.origintrail_noderc:/ot-node/.origintrail_noderc origintrail/ot-node:release_mainnet
```

If you are restoring a specific backup \(i.e. 4.1.17\), use the below:

```text
sudo docker create -i --log-driver json-file --log-opt max-size=1g --name=otnode -p 8900:8900 -p 5278:5278 -p 3000:3000 -v ~/.origintrail_noderc:/ot-node/.origintrail_noderc origintrail/ot-node:v4.1.17
```

### 8. Copy the restore script

```text
docker cp otnode:/ot-node/current/scripts/restore.sh ./
```

### 9. Run the restore script

```text
./restore.sh
```

![](../.gitbook/assets/image%20%2824%29.png)

