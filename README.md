# ansible-wireguard
ansible role for wireguard distribution

## Usaage
No need to store keys in files, just use 

`PRIVATE_KEY=$(wg genkey) && echo -ne "Private Key: $PRIVATE_KEY\nPublic Key: " && echo "$PRIVATE_KEY" | wg pubkey` 

to get keypairs. Create dedicated playbook for each wireguard mesh. Mesh is defined by `wireguard` complex list (example secret.yml) and `wg_group` string var.

!!! It is higly recommended to protect your `wireguard` list with ansible-vault !!!

Fields description:
* `name` - interface name for wireguard. will be `wg-kube` if name is `kube`
* `nodes` - FQDNs of your nodes, match with inventory hostname really needed here
* `node->wg_ip` - Internal IP address on wg interface (inside mesh)
* `node->wg_public` - Public key
* `node->wg_private` - Private key, may be skipped if it is just node peer configured manually
* `node->wg_port` - listen port, may be skipped for some peers as well (behid NAT mode)
* `node->wg_extip` - IP address to be exposed as part of `Endpoint` in config for peers.
* `node->wg_local` - Tag to separate zones. Nodes with the same wg_local will communicate directly with each other using wg_extip (LAN IPs) to build mesh. Nodes without wg_local will be avaliable for all peers. If some of your hosts behind NAT and, put them to the same wg_local. 
* `node->wg_route` - Route which will be added to peer configuration to `AllowedIPs` value.

### Tags for zone separation
Let's assume that we want to build a mixed network where some hosts are behind NAT (e.g. home lab), then for such hosts we can specify `home` in the `wg_local` field and use internal addresses (e.g. 192.168.x.y) as `wg_extip` . During the config generation phase it will be checked if the peers have the same tag, then their configs will specify `Endpoint` to internal addresses. This way hosts behind NAT can establish a connection directly within the local network. If `wg_local` is set to `global`, or not set at all, then the `wg_extip:wg_port` mapping will be set to `Endpoint` for all peers - this is the best option for hosts with a permanent public IP. 

### Problem with hosts behind NAT
The second problem that arises when building such mixed networks as mentioned above is one-way connection initiation. A host with a public IP cannot initiate a connection to a host behind NAT and has to wait, losing precious packets of traffic. Unfortunately, there is no more elegant and universal solution than adding a single line to crontab. Every 5 minutes all hosts send ICMP requests to all their peers. I would like to fix this in some other way.

