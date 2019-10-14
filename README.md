# Docker Zencash Secure Node

This repository will help you setup a zencash node with a single bash script.

The script will install Docker on a fresh Ubuntu VM and provision the following
containers:

- zend https://hub.docker.com/r/whenlambomoon/zend/
- Securenodetracker https://hub.docker.com/r/whenlambomoon/secnodetracker/

Certbot will be installed and auto-renew your SSL certificates when required.

## Requirements

You will need a server with at least 4GB ram (2GB memory w/ 2GB swap is generally OK) to run a ZenCash secure node. Common providers can be found here: https://horizenofficial.atlassian.net/wiki/spaces/ZEN/pages/136872080/Community+VPS+List

The script is designed to run on Ubuntu 16.04

You will also need a domain name that is pointing to your server (eg. zennode.example.com)
certbot will be used to provision and maintain a valid SSL certificate for your domain.

## Installation

Invoking the script is best done on a fresh installation, however executing install script again should not
cause any issues.

*Note:* Check the [ansible installer](https://github.com/sebbourgeois/docker-zen-node/tree/master/ansible) if you plan on installing multiple nodes

```
curl -O https://raw.githubusercontent.com/sebbourgeois/docker-zen-node/master/install.sh
chmod +x install.sh
./install.sh <stakeaddr> <email> <fqdn> <region> <nodetype>
```

- Stakeaddr is the transparent address which has your 42 ZEN
- Email is your notification email address should your node have issues
- FQDN is the hostname which must point to the IP address of your server
- Region must either be eu or na
- Nodetype is secure or super

Example:

`./install.sh ztjcr2DSYhMZZ3WFFeoK2hDxhmK4VP3QcgK email@example.com zennode.example.com na secure`

The script will install the required dependencies and deploy the three containers on your host.

The script should execute successfully when you see the output similar to this:

```
## Generating shield address for node...

ztSpgr2Yat7zSMiLNHtyDKGajzSesxabQsRwJnqtomDJU9wd6LzZppnQJyYiNE8sJDEy5MyTiMrSjf3bWcMKgtF9xcEY4eA
```

This will be the shield address you need to send 0.05 ZEN in order for your start accepting challenges. It is recommended you send this in 3 transactions of 0.0167 ZEN. As of now, this can only be done using the swing wallet or CLI.

After you send your 0.05 ZEN, you may check your node private balance:

```
docker exec zen-node gosu user zen-cli z_gettotalbalance
```

The securenodetracker will only report a successful status after you've sent your ZEN to the shield address. You may check by viewing the secnodetracker logs:

```
docker logs zen-secnodetracker
```

It should return something like this:

```
Secure node config found OK - linking...
CPU Intel Core Processor (Haswell, no TSX)  cores=1  speed=2394
Tracker app version: 0.1.0
MemTotal: 3.86GB  MemFree: 1.61GB  MemAvailable: 3.47GB  SwapTotal: 4.10GB  SwapFree: 4.10GB  
2017-12-01 07:29:58 GMT -- Connected to server ts1.na. Initializing...
2017-12-01 07:30:08 GMT -- Zend: Loading block index...
2017-12-01 07:30:18 GMT -- Zend: Verifying blocks...
Secure Node t_address (not for stake)=XXXX
Balance for challenge transactions is 1
Using the following address for challenges
xxxx
```

Finally, check the securenode tracker website to see if your node has been registered:

- NA: https://securenodes.na.zensystem.io/
- EU: https://securenodes.eu.zensystem.io/

**Please Note:** It may take a few hours for your server to fully update to the latest block. Your node will not come online until this has been fully updated.

You may check the latest block status with the following command:

```
docker exec zen-node gosu user zen-cli getinfo
```

## Adding new nodes

If you are creating multiple nodes, you may seed the initial blockchain from one of your running nodes with the following command:

```
rsync -avh --progress /mnt/zen/config/blocks /mnt/zen/config/chainstate /mnt/zen/config/database yournewnode:/mnt/zen/config/
```

This will save you the 1-2 hours wait for a new node to download the full chain.

## Moving/removing nodes

If you want to move the ZEN off the node you need to run the following commands:

```
# Get your T_ADDR
docker exec -it zen-node gosu user zen-cli z_listaddresses

# Get the balance of your T_ADDR
docker exec -it zen-node gosu user zen-cli z_getbalance T_ADDR

# Send the balance to another address
docker exec zen-node gosu user zen-cli z_sendmany "T_ADDR" "[{\"amount\": 0.00, \"address\": \"TO_ZEN_ADDR\"}]"
```

Remember to deduct the 0.0001 fee from your balance if you wish to empty the entire balance.

## Troubleshooting

If you need to execute commands with `zen-cli` you will need to append the following:

```
docker exec zen-node gosu user zen-cli
```

This is because zend is run as a non-root user using `gosu`. Removing `gosu user` will return
you an error due to missing config files.

If you run into any issues feel free to open an issue. Please include output from the following commands when opening your issue:

```
docker logs zen-secnodetracker
docker logs zen-node
```

If your zen-node becomes corrupted due to lack of disk space you may see an error similar to this:

```
2017-12-10 21:22:46 Corruption: checksum mismatch
2017-12-10 21:22:46 : Error opening block database.
Please restart with -reindex to recover.
2017-12-10 21:22:46 Aborted block database rebuild. Exiting.
2017-12-10 21:22:46 scheduler thread interrupt
2017-12-10 21:22:46 Shutdown: In progress...
2017-12-10 21:22:46 StopRPC: waiting for async rpc workers to stop
2017-12-10 21:22:46 StopNode()
2017-12-10 21:22:46 Shutdown: done
```

To resolve this you will need to reindex your zen-node. To do this you will need to run the following commands:

```
systemctl stop zen-node
docker run --rm --net=host -p 9033:9033 -p 18231:18231 -v /mnt/zen:/mnt/zen -v /etc/letsencrypt/:/etc/letsencrypt --name zen-node whenlambomoon/zend:latest zend -reindex
```

Once it finishes reindexing you can exit and restart the zen-node normally:

```
docker stop zen-node
systemctl restart zen-node
```

## Donations

Donations are appreciated:

**ETH** 0x19B41e42de7696D25761B6966eF6113585a3b034

**ZEN** t1Rj2tMj5eqZpiNusGFEDtQz1X5dmsAfzqE

**BTC** 3CUHykuHkP1d3YdmdxBDpmtdsDfLALZSds
