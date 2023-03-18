#### Deploy
```
cd 4-node/
sudo containerlab deploy -t c8201-4-node.yml 
```
output:
```
INFO[0000] Containerlab v0.33.0 started                 
INFO[0000] Parsing & checking topology file: c8201-4-node.yml 
INFO[0000] Creating lab directory: /home/brmcdoug/vxr8000-containerlab/4-node/clab-c8201-4-node 
INFO[0000] Creating docker network: Name="clab", IPv4Subnet="172.20.20.0/24", IPv6Subnet="2001:172:20:20::/64", MTU="1500" 
INFO[0000] Creating container: "r04"                    
INFO[0000] Creating container: "r03"                    
INFO[0000] Creating container: "r02"                    
INFO[0000] Creating container: "r01"                    
INFO[0004] Creating virtual wire: r03:FH0_0_0_2 <--> r04:FH0_0_0_2 
INFO[0004] Creating virtual wire: r03:FH0_0_0_3 <--> r04:FH0_0_0_3 
INFO[0005] Creating virtual wire: r01:FH0_0_0_2 <--> r03:FH0_0_0_0 
INFO[0005] Creating virtual wire: r01:FH0_0_0_3 <--> r03:FH0_0_0_1 
INFO[0005] Creating virtual wire: r02:FH0_0_0_3 <--> r04:FH0_0_0_1 
INFO[0005] Creating virtual wire: r01:FH0_0_0_0 <--> r02:FH0_0_0_0 
INFO[0005] Creating virtual wire: r02:FH0_0_0_2 <--> r04:FH0_0_0_0 
INFO[0005] Creating virtual wire: r01:FH0_0_0_5 <--> r04:FH0_0_0_5 
INFO[0005] Creating virtual wire: r01:FH0_0_0_4 <--> r04:FH0_0_0_4 
INFO[0005] Creating virtual wire: r01:FH0_0_0_1 <--> r02:FH0_0_0_1 
INFO[0006] Adding containerlab host entries to /etc/hosts file 
+---+-----------------------+--------------+----------------+-------+---------+----------------+----------------------+
| # |         Name          | Container ID |     Image      | Kind  |  State  |  IPv4 Address  |     IPv6 Address     |
+---+-----------------------+--------------+----------------+-------+---------+----------------+----------------------+
| 1 | clab-c8201-4-node-r01 | 0661841b86f7 | 8201clab:7.5.3 | linux | running | 172.20.20.5/24 | 2001:172:20:20::5/64 |
| 2 | clab-c8201-4-node-r02 | 60c1c55f0890 | 8201clab:7.5.3 | linux | running | 172.20.20.4/24 | 2001:172:20:20::4/64 |
| 3 | clab-c8201-4-node-r03 | 9411408c325e | 8201clab:7.5.3 | linux | running | 172.20.20.3/24 | 2001:172:20:20::3/64 |
| 4 | clab-c8201-4-node-r04 | 13a406c70bd8 | 8201clab:7.5.3 | linux | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |
+---+-----------------------+--------------+----------------+-------+---------+----------------+----------------------+
```
```
docker logs -f clab-c8201-4-node-r04

```
