# DIGITALPRICE-TO-TEST
DigitalPriceMasternode

Step-by-step guide to setup a masternode in a Linux VPS

Requirements

- DigitalPrice wallet running in your local computer with at least 25000 DPs.
- The software Putty to connect and send commands through SSH.
- The software WinSCP to see your VPS's folders, it will ease the configuration.
- A VPS running a Linux distribution.

## Step 1: Compiling the DigitalPrice wallet on your VPS

- Connect to your VPS with the Putty.
- Download the wallet to be compilled, with the commands below. Remember the folder where the DigitalPrice will be stored. 
```
sudo apt-get install git
git clone https://github.com/cpozzer/DigitalPrice
```
- Install the dependencies:
```
sudo apt-get -y update && sudo apt-get -y install build-essential libssl-dev libdb++-dev libboost-all-dev libcrypto++-dev libqrencode-dev libminiupnpc-dev libgmp-dev libgmp3-dev autoconf autogen automake libtool
```
- Access your DigitalPrice folder and run the following command:
```
cd src
sudo make -f makefile.unix 
```
Your VPS will start compiling the wallet. This operation can take more than 1 hour on a Raspberry PI so be patient. If your VPS lacks of RAM to compile, follow this guide to add memory using a Swap file. 
https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-16-04
After finished, you should see a new file named "digitalpriced" in your digitalprice/src folder.

## Step 2: Starting and Configuring the wallet

- Start the wallet typing the following command in the src folder: 
```
./digitalpriced -daemon
```
- Your should get "DigitalPrice server starting". 
When you start the wallet for the first time a new folder is created with the chain, the conf file and so in. This folder is located in the same repository as the DigitalPrice one. For example if your DigitalPrice folder path is /home/digitalprice , then the new folder path is /home/.dprice (sometimes its located in the /root folder). 
You won't see it in WinSCP so you need to make hidden folders visible by clicking on the small icon at the bottom right of WinSCP.
- Now you need to configure your wallet, first we need to close the wallet and edit the conf file:
```
./digitalpriced stop 
cd ~/.dprice 
sudo nano digitalprice.conf 
```
The file should look like this: 
```
rpcuser=Random_string 
rpcpassword=Longer_random_string 
rpcallowip=127.0.0.1 
listen=1 
server=1 
daemon=1 
staking=0 
```
- Save the file, restart the wallet and wait it to get fully synced. 
```
cd .. 
cd digitalprice/src 
./digitalpriced 
```
You can check the progress of the syncronization of the wallet by typing this command: 
```
./digitalpriced getmininginfo
```
## In this guide, we will show how to configure your masternodes to be controlled remotely with a cold wallet, since it's easier and safer.

- Open your Windows wallet, go into the console and type: 
```
masternode genkey
```
- Copy the result in a file. This is your masternode private key. You will need to use this string later.
- Create a new address to receive your 25000 DPs, typing: 
```
getaccountaddress mn01
```
- Send exactly 25000 DP to the address you just created and wait for 10 confirmations. You have to close the wallet now. It will be reopen later.
- Now you need to edit the digitalprice.conf of your local wallet. 
For Windows users the file is located in C:\Users\your_name\AppData\Roaming\dprice. 
To see this folder you need to make hidden folders visible. 
If you do not have the digitalprice.conf file in your folder, create a txt file name "digitalprice.txt" then change the extension to digitalprice.conf. We need to edit this file, just copy:
```
rpcuser=SomeRandomString 
rpcpassword=EvenLongerRandomString 
rpcallowip=127.0.0.1 
listen=1 
server=1 
daemon=1 
staking=0 
logtimestamps=1 
```
- You will need to edit the digital.conf on your VPS again. Before, the wallets needs to be closed and fresh (no transactions). Just type:
```
./digitalpriced stop
```
- Open the digitalprice.conf in your VPS, located in the .dprice folder. Fill the file as below:
```
rpcuser=SomeRandomString 
rpcpassword=EvenLongerRandomString 
rpcallowip=127.0.0.1 
listen=1 
server=1 
daemon=1 
staking=0 
logtimestamps=1 
port=9999 
masternode=1 
masternodeaddr=XXX.XXX.XXX.XXX:9999 
masternodeprivkey=XXXXXXXXXXXXXXXXXXXXXXXXXX 
```
- This one must be exactly the same as the digitalprice.conf of your local wallet.
```
port: Select an open port, I recommend you to use 9999 
masternodeaddr: You have to put your VPS ip and the port. 
masternodeprivkey: It's the string you got from the "masternode genkey" command before. 
```
- Save the file. In the same folder
- Delete your wallet.dat (if it's an empty wallet there is no problem, else backup it then delete), the txlvldb folder and the blk00001.dat using WinSCP.
- Keep WinSCP open, we gonna copy some files. 
- Go back to your Windows dprice folder where you edited the first digitalprice.conf and drag the txlvldb folder and the blk00001.dat file to your .dprice VPS folder. 
It will take 3-4 minutes to copy. When its done, we can start again the VPS wallet. 
If your DigitalPrice folder path is /home/digitalprice then:
```
cd /home/digitalprice/src 
./digitalpriced 
```
- Open your local wallet again. Go to the Debug console and type:
```
masternode outputs
```
A string will be printed, something like that : 
```
"aa7c6c173f7b691e5a070a37aeazd23557636ad1b4b43680ace39d522e1d4493" "1"
``` 
The first part is your transaction hash, the "1" is the index. 
- Save them to be used next.
Now we gonna create the masternode.conf file. 
- In your wallet, click on the "Masternodes" tab, you will see the list of active masternodes. 
- Click on My masternodes and on Create. A box appears:
```
Alias : The name of your masternode, type mn01. 
Address : The IP and the port you used in your VPS conf file, use the same. 
Privkey : Your masternode privkey from the "masternode genkey" command. 
TxHash : The first part of the "masternode outputs" command, in our example it's aa7c6c173f7b691e5a070a37aeazd23557636ad1b4b43680ace39d522e1d4493 
OutputIndex : The last number, in our example it's 1. 
```
- Click Ok and wait few seconds. A new entry is created, its your masternode. It should say that your masternode is not on the list. To start it, unlock the wallet, press start and you are done!
