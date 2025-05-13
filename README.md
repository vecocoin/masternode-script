# Veco DIP3 Masternode Installation Guide

This guide will teach you how to setup a Veco DIP3 Masternode on a remote server (VPS). You should have at least a basic knowledge of linux. For better clarity, all commands that must be typed shall be displayed as such:
```bash
this is a command
```
Type the command exactly (you may copy&paste it). There will always be some space between commands so that you can easily see commands spanning over several lines. Some commands may also be appended together with **&&** to speed up the process when commands are short or trivial. You may execute these commands as one, or you may type each one separately.

If you need any additional help, feel free to join [our discord](https://discord.gg/JRzSRYF7) and ask for help in the _#masternodes_ channel.

**BEWARE scammers trying to impersonate team members! Do not accept help from people directly contacting you. No one from Veco team will contact you and “help” proactively!**

## What you’ll need

1. **A local computer** – your everyday (typically Windows) computer, which will run a control wallet and hold your coins. This computer need not always be online. 

2. **A remote (or local) server** – typically a VPS, or a server at home. You can choose a 64-bit OS; Ubuntu Server 18.04, 20.04 or 22.04. You need a unique and static IP address (an IP address that does not change), which is always running and connected to the Internet. This server should at least have 1Gb of RAM and 10Gb of space storage.
(When you want to install multiple masternodes on one machine, you should add 750 Mb of RAM en 2 Gb of space storage to this for every extra masternode.)

3. **A collateral** – an amount in Veco that will be unspendable as long as you wish to keep your node running. For a masternode you’ll need 10,000 VECO. You’ll need some change for transaction fees, so 1 VECO more to cover expenses is good enough.

## Setting up a Control wallet

### Step 1 – Set up a wallet

This involves downloading and synchronizing the [wallet](https://github.com/vecocoin/veco/releases/download/v1.14.1/veco-1.14.1-win64-setup.exe). Please use a [bootstrap](https://github.com/vecocoin/veco/releases/tag/v1.14.1.bs) to speed up the process.

### Step 2 – Create collateral

As mentioned above, you will need some Veco to create what is called collateral: a certain amount of Veco that will be “frozen” in order for your masternode to keep running.

You will first need to get the amount of Veco for the collateral, as well as a small amount to pay the transactions fees. 
You may purchase some Veco on exchange [FreiXLite](https://freixlite.com/market/VECO/LTC) . You will need:

- 10,000 VECO (+1 VECO) for a masternode

Once you have this amount in your control wallet, you need to set it as official collateral. This is done by creating a receiving address in your wallet, and sending the exact amount – 10,000 VECO – to it. Making a payment to yourself requires a fee. This is why you needed that extra VECO.

After making that payment, you will need to retrieve some information about it: the Transfer ID and Index.

This information can be found using the Debug Console. Go to Tools > Debug Console. Type the following command:
```bash
masternode outputs
```
this will yield something like this:

{  
"efa598dd5df8fdff8777b1bf36066bbda34426a2bba33c702867d67e64070707": "1"  
}

The first part is the transfer ID, and the last (the digit) is the index.

### Step 3 – Create private key and BLS key

A private key was used to identify your masternode in the DIP1 control wallet. The wallet is doing a check for it, so you still need it. 
You can create this key using the console again and typing the following command:
```bash
masternode genkey
```
This key will work for masternode.

The BLS key is needed to prepare for the new DIP3 deterministic masternodes. You may read more about deterministic masternodes [here](https://blog.dash.org/introducing-deterministic-masternode-lists-daaa7c9bef34).
```bash
bls generate
```
The command yields a secret key and a public key. You will need to keep both parts for future use. The secret key will be inserted in the veco.conf file of the masternode and the public key will be needed to activate the masternode on the blockchain once the DIP3 protocol is activated.

## Setting up a VPS

The following procedure assumes an installation from scratch. If you have an existing VPS already installed, then some steps might not be needed. 


### Step 1 – Acquire a VPS from any provider

The cheapest one will do, provided you create a swap file when RAM is not sufficent. (see below).
We recommend Ubuntu 22.04, but when not available you could choose the Ubuntu  18.04 or 20.04 LTS Linux distribution.

### Step 2 – Log into your VPS and install updates and take security measurements


In order to access your VPS, you will need a software/SSH client such as [PuTTY](https://www.putty.org/). This tutorial does not cover installation of, know-how to use such software.

Once you have access to your VPS, create a user that will be running your masternode (for security reasons, it is always better not to run any application as root user):
```bash
adduser veco && adduser veco sudo
```
This creates user **veco** with root privileges (be able to run root commands using the “sudo” prefix).

Switch to **veco** user
```bash
su veco
cd ~
```

A clean server install will likely need some software updates. Enter the following command which will bring the system up to date (can take a few minutes to complete):
```bash
sudo apt-get update && sudo apt-get upgrade -y
```
**BEWARE: Securing your server is very important and your responsibility!** 
To secure your VPS, immediately change default passwords, update all software, configure a firewall to allow only necessary ports, use SSH keys instead of passwords for remote access, and consider installing fail2ban to block brute-force attempts. 
For detailed instructions, you can refer to this comprehensive guide: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-22-04 1 

Reboot your VPS for changes to take effect:
```bash
sudo reboot
```
After rebooting, switch to **veco** user again:
```bash
su veco
cd ~
```

### Step 3 – Set up a Swap File (optional)

This will be needed especially if using a low end VPS and you wish to compile the source code. 

Some providers already install a swap on their VPS. You can check this by doing:
```bash
htop
```
This provides you with a nice view of your VPS resources. In the higher left part, check if **Swp** has any value higher than **0K**. If so, you are good to go to Step 4. If not, continue below.

The following command sets up a 2GB swap file. You may change this size by modifying the **2G** to anything you like (we still recommend at least **2G**). Leave all other commands unchanged.
```bash
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile && sudo cp /etc/fstab /etc/fstab.bak && echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
[More information about swap files](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-18-04)

### Step 4 – Install masternode binaries and configuration

Get the latest binaries [from github](https://github.com/vecocoin/veco). At the time of writing, latest version is **v1.14.1**. 

You should check on github and adapt the following commands with **latest binaries** and **Ubuntu version** reference.

Example for **v1.14.1** and **Ubuntu 22.04 ** VPS:
```bash
wget https://github.com/vecocoin/veco/releases/download/v1.14.1/veco-1.14.1-ubuntu_2204_x86.tar.gz
```
```bash
tar zxvf veco-1.14.1-ubuntu_2204_x86.tar.gz

sudo mv ~/src/veco{d,-cli,-tx} /usr/local/bin/

rm -r vecocore-1.14.1
```
Before the node can operate as a masternode, a custom configuration file needs to be created. Since we have not loaded the blockchain yet, we will create the necessary directory and configuration file
```bash
mkdir .vecocore && cd .vecocore
```
Get the following values for your configuration file:

-   **USER** – an alphanumerical string, e.g. "VecoUser"
-   **PASSWORD** – an alphanumerical string, not the same as user, e.g. "VecoPasswd"
-   **VPS IP** – the IP of your VPS (looks something like: 11.22.33.44)
-   **PRIVATE KEY** – the one you created earlier in your control wallet’s Debug Console
-   **BLS SECRET KEY** – the one you created earlier in your control wallet’s Debug Console

When you want to install multiple masternodes on one machine, use the supplied IPv4 address for the first masternode, and use IPv6 addresse for the next masternodes.
Look for the information supplied by the provider about IPv6 addresses. With most providers you need to take some extra actions to activate the IPv6 addresses. 

Create _veco.conf_ file
```bash
nano veco.conf
```
then copy&paste the following in it, inserting the proper values:
```bash
rpcuser=USER
rpcpassword=PASSWORD
listen=1
server=1
daemon=1
maxconnections=50
masternode=1
externalip=VPS IP
rpcallowip=127.0.0.1
rpcport=26920
masternodeprivkey=PRIVATE KEY
masternodeblsprivkey=BLS SECRET KEY
```

Save the file (**ctrl-x**, then type **y** and hit **enter**)

### Step 5 – Start the daemon

Now that you have everything set up, it’s time to start the daemon. To speed up the synchronization of the blockchain, you may download a booststrap file and put it in the .vecocore directory you created earlier, the one where your veco.conf file was created (this is highly recommende to prevent long delays):
```bash
wget https://github.com/tedydet/veco/releases/download/bootstrap/bootstrap_1.14.1.zip && unzip bootstrap_1.14.1.zip
```
And finally launch the masternode daemon:
```bash
vecod -daemon
```
Wait about 30 minutes for your masternode to sync completely.

You may monitor the sync progress using the following command:
```bash
watch veco-cli getinfo
```
which should yield the following information:

```
{
  "version": 1140100,
  "protocolversion": 70211,
  "walletversion": 61000,
  "balance": 0.00000000,
  "privatesend_balance": 0.00000000,
  "blocks": 1406426,
  "timeoffset": 0,
  "connections": 8,
  "proxy": "",
  "difficulty": 0.00210908149541103,
  "testnet": false,
  "keypoololdest": 1746505472,
  "keypoolsize": 999,
  "paytxfee": 0.00000000,
  "relayfee": 0.00001000,
  "errors": ""
}
```
Here, “watch”-ing lets you see the synchronization (you can exit the watch at any time with **ctrl-c**). The blocks number will go up until your masternode reaches the total number of blocks in the blockchain.

This is the longest part. You can see what number it needs to reach by hovering over the small **V** in the lower right of your control wallet or by checking [Veco Block Explorer](https://explorer.vecocoin.com).

**BEWARE: the blocks number might not start growing for a while. This is because the daemon could be looking for a valid connections or synchronizing headers. As long as you have connections higher than zero you are fine.**

You may verify that it is synced using the command:
```bash
watch veco-cli mnsync status
```
which should yield the following information:

```
{  
"AssetID": **999**,  
"AssetName": "MASTERNODE_SYNC_FINISHED",  
"AssetStartTime": 1650834730,  
"Attempt": 0,  
"IsBlockchainSynced": true,  
"IsMasternodeListSynced": true,  
"IsWinnersListSynced": true,  
"IsSynced": true,  
"IsFailed": false  
}
```

The masternode is completely synced when AssetID is **999** (it will go through 0, 1, 2, 3, 4 and 999).
You can exit the watch at any time with **ctrl-c**.
Once your masternode is synced, you may delete the bootstrap file:
```bash
rm bootstrap_1.14.1.zip
```

## Registration and starting the masternode

Your masternode is now synchronized and chatting with the network but is not accepted as a masternode because it hasn’t been introduced to the network with your collateral. This is done with the control wallet.

#### Step 1 – Activate masternodes tab

If the masternodes tab is not available (should be available by default) in your control wallet, you need to activate it. Go to Settings > Options, then Wallet tab and check the Show Masternodes Tab box.


#### Step 2 – Open control wallet

At last, we arrive to the final step needed to register the masternode. 

1.	Open your Veco control wallet and navigate to your console via Tools -> Debug console
   
#### Step 3 -	Generate two new addresses
*	Enter getnewaddress “label” (example: getnewaddress MN1)
*	Enter getnewaddress “label” (example: getnewaddress MN1-owner)
  
#### Step 4 -	Send collateral amount
Send exactly 10,000 VECO to the first address and wait until the transaction has 15 confirmations
   
#### Step 5 -	NX ID and TX Index	
Run masternode outputs in the console and note the respective TX-ID and TX-Index for the steps below

#### Step 6 -	Prepare Chain registration
Next you need to prepare the MN to be registered on chain using the template below (ie replace the bold variables with your data):

#### protx register_prepare collateralHash collateralIndex ipAndPort ownerKeyAddr operatorPubKey votingKeyAddr operatorReward payoutAddress feeSourceAddress

-	**collateralHash** = TX-ID of the transaction containing the 1000 VECO
-	**collateralIndex** = TX-Index of the transaction containing the 1000 VECO
-	**ipAndPort** = IP and p2p port (IPv4: 5.189.159.94:40000 | IPv6: [2a02:c207:3005:3682::19]:40000)
-	**ownerKeyAddr** = The second new address generated
-	**operatorPubKey** = Public Key from BLS keypair
-	**votingKeyAddr** = The second new address generated
-	**operatorReward** = 0
-	**payoutAddress** = Your MN address or a new one you want to receive the rewards
-	**feeSourceAddress** = An address in your wallet with few VECO for TX fees

If the command was executed successfully the wallet will respond with 3 strings containing **“tx“, “collateralAddress“, “signMessage“**

#### Step 7 -	Sign Message
Next run **signmessage collateralAddress signMessage** , replacing both variables with the data above (without “”)
If the command was executed successfully the wallet will respond with 1 string containing the signature

#### Step 8 -	Submit to chain
Next run **protx register_submit tx signature** to finally submit the masternode to the chain
You do not need to keep a record of that console output. After a few blocks you should see your new masternode appear in the masternode tab of your controller with status "ENABLED"

Note: There is no more masternode.conf file or similar with DIP 3 masternodes. Everything is done “on-chain” using the commands above. 

#### Step 9 -	Verification of status
You can verify your masternode is running successfully on VPS with the following command (as veco user):
```bash
veco-cli masternode status
```
If the state “READY” is displayed, your masternode is running and you will get your reward on the Next Payment block.

