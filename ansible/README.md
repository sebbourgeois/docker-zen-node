# Ansible Installer

The ansible installer will allow you to configure multiple nodes simultaneously.

If you have an existing node it will be easiest to run the installer from that node.

```
apt-get install ansible
git clone https://github.com/sebbourgeois/docker-zen-node/
cd docker-zen-node/ansible/
```

Configure the hosts files with your list of nodes, an example hosts file would look like this:

```
# Parameters passed to all hosts
[all:vars]
email=test@example.com
region=na

[zen-nodes]
node.example.com fqdn=node.example.com stakeaddr=test nodetype=secure
node2.example.com fqdn=node3.example.com stakeaddr=test nodetype=secure
node3.example.com fqdn=node2.example.com stakeaddr=test nodetype=secure
```

If you are running this from an existing node, jump down to the [bootstrap node](https://github.com/sebbourgeois/docker-zen-node/tree/master/ansible#bootstraping-the-blockchain) section
to speed up node installation.

Now run the installer, this will install the securenodes on all of your listed hosts.

```
ansible-playbook -i hosts main.yml
```

When the installation completes, all your installed nodes details will be stored in /tmp/zen-node-results

```
cat /tmp/zen-node-results
```

It will list results similar to this:

```
localhost:

Shield Address:
zcNBdJxZnhTZMdiSQABYiW1wY2A8Swrq8TsauYLiyaShKD91GrZvn1dqAkhZ8USmAHoKNxhokeoYJZwJAtKjyeWN4BMNM6v

Balance:
  transparent: 0.00
  private: 0.00
  total: 0.00
```

You will now need to send the 1 ZEN to the shield addresses that have a balance of 0.0

### Upgrading

The installer has a handy way to update all of your zen-nodes:

```
ansible-playbook -i hosts upgrade.yml
```

### Status

You may check the status of your nodes anytime with the command:

```
ansible-playbook -i hosts status.yml
```

This will report the node name, shield address, balance and current block.
