# Multichain setup in Ubuntu
It should work for any debian based setup

## Introduction
This is a guide for installing multichain on at least two nodes with access on `10.0.0.0/24` subnet (or whatever private network you want).
In my setup I use Hetzner Cloud and keep the nodes in different datacentres linked with a private network and only accessible from my own workstation to the outside world.
In this tutorial we will create the QMSchain blockchain to go with the QMS block chain github repo

## Step 1: Install Multichain on Both Nodes
### 1.1. Update your system:
```bash
sudo apt update
sudo apt upgrade -y
```

### 1.2. Install required dependencies:
```bash
sudo apt install -y wget tar
```

### 1.3. Download and install Multichain:
You need to download the Multichain binary package from the official website.

The curent community edition is available [here](https://www.multichain.com/download-community/)

run this as a super user
``` bash
su
```

```bash
cd /tmp
wget https://www.multichain.com/download/multichain-2.3.3.tar.gz
tar -xvzf multichain-2.3.3.tar.gz
cd multichain-2.3.3
mv multichaind multichain-cli multichain-util /usr/local/bin
```

```bash
exit
```

Now, Multichain is installed on the system.

## Step 2: Create the Blockchain (on Node 1)
### 2.1. On the first node, initialize the blockchain:
```bash
multichain-util create QMSchain
```
Replace `QMSchain` with your desired blockchain name.

### 2.2. Configure the blockchain parameters:
Edit the configuration file (optional at least have a look through it):

```bash
nano ~/.multichain/mychain/params.dat
```

You can adjust parameters such as maximum block size, mining rewards, or transaction fees.

### 2.3 Start the blockchain on Node 1:

```bash
multichaind mychain -daemon
````

After running this, the blockchain will be started and listening for incoming connections. The console will provide a "connect string" that you’ll need for the second node, e.g.: `multichaind mychain@<ip>:<port>` **Copy this connect string for later use.**

## Step 3: Connect Node 2 to the Blockchain
### 3.1. Install Multichain on Node 2:

Follow the same installation steps from Step 1 on the second node.

### 3.2. Connect Node 2 to the blockchain:
On Node 2, use the connect string provided by Node 1: `multichaind mychain@<ip>:<port>`

This will download the blockchain's metadata from Node 1 and start synchronizing the chain. Once connected, it will show the blockchain activity on the console.

## Step 4: Allow Permissions (on Node 1)
Multichain uses a permissioned model, so you need to grant permissions to Node 2 to allow it to participate in the chain.

### 4.1. Get the address of Node 2:

On Node 2, run:
```bash
multichain-cli mychain getaddresses
```
This will return the node's address. Copy this address for the next step.

### 4.2. Grant permissions to Node 2:

On Node 1, grant the connect and send permissions to Node 2:
```bash
multichain-cli mychain grant <node2-address> connect,send,read
```

## Step 5: Verify the Connection
### 5.1. Check the connected nodes on Node 1:
```bash
multichain-cli mychain getpeerinfo
```
This will show information about Node 2 being connected.

### 5.2. Verify blockchain status:
On both nodes, you can check the blockchain status by running:
```bash
multichain-cli mychain getinfo
```
This command will display details about the chain, blocks, and connections.

## 6. Remote RPC Access to a Node
**You only need to do this on one or a few (if you have lots) of nodes**
Likely you want to do something with your nice shiny new blockchain.
A server can push transactions to the blockchain by interacting with a full node (either a remote or local node) using Multichain's RPC (Remote Procedure Call) interface.

**Pros:** This method doesn’t require the server to run a full node itself but still allows it to interact with the blockchain using the multichain-cli commands or direct HTTP calls to the RPC interface.
**Cons:** Requires access to the RPC credentials and permissions on the node it connects to.

Steps for RPC Access:

### 6.1. Enable RPC on the Multichain node: Edit the `multichain.conf` file (usually in `~/.multichain/mychain/`) and add:

```bash
rpcuser=multichainrpc
rpcpassword=<secure_password>
rpcallowip=<server-ip>
rpcport=port_number (optional)
```

### 6.2. Restart the node to apply changes:
```bash
multichaind mychain -daemon
```

### 6.3. Send an RPC request from the server: 
You can use tools like curl to interact with the node’s RPC interface directly from the server:
```bash
curl --user multichainrpc:<secure_password> --data-binary \
'{"method":"send","params":["<address>",10],"id":1}' \
-H 'content-type: text/plain;' \
http://<node-ip>:port_number
```

## 7.0 setting up multichain as a system service:
Most likley you want the multichain to start on reboot so you need to setup a system service to do this. 

### 7.1: Edit the MultiChain Service File
Edit the systemd service file:

```bash
sudo nano /etc/systemd/system/multichain.service
```

```ini
[Unit]
Description=MultiChain node
After=network.target

[Service]
ExecStart=/usr/local/bin/multichaind QMSchain
User=root
Group=root
Restart=always

[Install]
WantedBy=multi-user.target
```

*Note its not advisible to always run as root maybe you want to make a multichain group?*

### 7.2 Enable and Start the Service:

On both nodes, run:
```bash
sudo systemctl enable multichain.service
sudo systemctl start multichain.service
```

Step 8: Test and Start Using the Blockchain
After setting everything up, test transactions or asset creation on either node to ensure both are syncing and communicating.

**You now have a basic two-node MultiChain network running!**


