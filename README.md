# vxr8000-containerlab

### Installing and running Cisco 8000 simulator Containerlab docker image

1. Host server requirements

- verify that Open vSwitch is installed
```
ovs-vsctl --version
```
- limit /proc/sys/kernel/pid_max to 1048575
```
sysctl -w kernel.pid_max=1048575
```
- make sure KVM modules are installed and configured
  file /dev/kvm (the output should be /dev/kvm: character special)

- cpu and memory: recommend 20GB and 4 cores per router

2. Install 8000 docker image
```
docker load -i 8201clab:7.5.3.tar.gz
```
3. Create a topology file, example: [c8201-b2b.yml](c8201-b2b.yml)


#### NOTE: Data interfaces naming convention 
```
FH0_0_0_[0..N] ->  400G interfaces
Hu0_0_0_[0..N] ->  100G interfaces
```
4. Create config files. Examples: [R01](r01.cfg) and [R02](r02.cfg)

5. Deploy topology
```
sudo containerlab deploy -t c8201-b2b.yml
```
Example terminal output:
```
INFO[0000] Containerlab v0.33.0 started                 
INFO[0000] Parsing & checking topology file: c8201-b2b.yml 
INFO[0000] Creating lab directory: /home/brmcdoug/vxr8000-containerlab/clab-c8201 
INFO[0000] Creating docker network: Name="clab", IPv4Subnet="172.20.20.0/24", IPv6Subnet="2001:172:20:20::/64", MTU="1500" 
INFO[0000] Creating container: "r02"                    
INFO[0000] Creating container: "r01"                    
INFO[0003] Creating virtual wire: r01:FH0_0_0_0 <--> r02:FH0_0_0_0 
INFO[0003] Creating virtual wire: r01:FH0_0_0_1 <--> r02:FH0_0_0_1 
INFO[0004] Adding containerlab host entries to /etc/hosts file 
+---+----------------+--------------+----------------+-------+---------+----------------+----------------------+
| # |      Name      | Container ID |     Image      | Kind  |  State  |  IPv4 Address  |     IPv6 Address     |
+---+----------------+--------------+----------------+-------+---------+----------------+----------------------+
| 1 | clab-c8201-r01 | f6d57e4fe0dd | 8201clab:7.5.3 | linux | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |
| 2 | clab-c8201-r02 | 9e9c97bc8278 | 8201clab:7.5.3 | linux | running | 172.20.20.3/24 | 2001:172:20:20::3/64 |
+---+----------------+--------------+----------------+-------+---------+----------------+----------------------+
```
NOTE: it takes about 6 minutes for XR to come up.


6. Monitor sim bringup
```
docker logs -f clab-c8201-r01
...
6:56:33 INFO Getting port files for device R0
16:57:03 INFO R0:wait for XR login prompt (console output captured in vxr.out/logs/console.R0.log)

After a few minutes you should see:

...
17:01:28 INFO R0:applying XR config
17:01:29 INFO Sim up
Router up
```
If you don't see "Router up" after 6-10 minutes, please share the "docker logs ..." output with Cisco team.


7. Test ssh to XR (use cisco/cisco123 login credentials)
```
sudo containerlab inspect -t c8201-b2b.yml |grep r01

INFO[0000] Parsing & checking topology file: c8201-b2b.yml 
| 1 | clab-c8201-r01 | f6d57e4fe0dd | 8201clab:7.5.3 | linux | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |

ssh cisco@172.20.20.2
Password:


RP/0/RP0/CPU0:r01#show platform 
Fri Dec  9 21:27:36.450 UTC
Node              Type                     State                    Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        8201-32FH(Active)        IOS XR RUN               NSHUT
0/PM0             PSU2KW-ACPI              OPERATIONAL              NSHUT
0/PM1             PSU2KW-ACPI              OPERATIONAL              NSHUT
0/FT0             FAN-1RU-PI-V2            OPERATIONAL              NSHUT
...


RP/0/RP0/CPU0:r01#show version
Fri Dec  9 21:28:08.453 UTC
Cisco IOS XR Software, Version 7.5.3 LNT
Copyright (c) 2013-2022 by Cisco Systems, Inc.

Build Information:
 Built By     : ingunawa
 Built On     : Tue Sep 27 08:57:08 UTC 2022
 Build Host   : iox-ucs-031
 Workspace    : /auto/srcarchive16/prod/7.5.3/8000/ws
 Version      : 7.5.3
 Label        : 7.5.3-iso

cisco 8000 (VXR)
cisco 8201-32FH (VXR) processor with 16GB of memory
r01 uptime is 15 minutes
Cisco 8201-32FH System
```

8. Run ping test & check ISIS adjacency
```
RP/0/RP0/CPU0:r01#show ipv4 int br
Fri Dec  9 21:28:38.638 UTC

Interface                      IP-Address      Status          Protocol Vrf-Name
Loopback0                      10.0.0.1        Up              Up       default 
FourHundredGigE0/0/0/0         10.1.1.0        Up              Up       default 
FourHundredGigE0/0/0/1         10.1.1.2        Up              Up       default 
FourHundredGigE0/0/0/2         unassigned      Shutdown        Down     default 
...

RP/0/RP0/CPU0:r01#ping 10.1.1.1 
Fri Dec  9 21:29:12.153 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 10/12/20 ms
RP/0/RP0/CPU0:r01#ping 10.0.0.2
Fri Dec  9 21:29:20.166 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/12/20 ms
RP/0/RP0/CPU0:r01#show isis adj
Fri Dec  9 21:29:27.677 UTC

IS-IS 100 Level-1 adjacencies:
System Id      Interface                SNPA           State Hold Changed  NSF IPv4 IPv6
                                                                               BFD  BFD 

IS-IS 100 Level-2 adjacencies:
System Id      Interface                SNPA           State Hold Changed  NSF IPv4 IPv6
                                                                               BFD  BFD 
r02            FH0/0/0/0                *PtoP*         Up    23   00:09:58 Yes None None
r02            FH0/0/0/1                *PtoP*         Up    29   00:09:58 Yes None None

Total adjacency count: 2
RP/0/RP0/CPU0:r01#
```

XR serial console access
========================


1. Telnet to serial console (port 60000)
```
docker exec -ti clab-c8201-r01 telnet 0 60000
Trying 0.0.0.0...
Connected to 0.
Escape character is '^]'.

RP/0/RP0/CPU0:r01#
```

Troubleshooting
===============


1. Invoke bash shell of the relevant docker instance
docker exec -ti clab-c8201-r01 bash
root@r01:/#

2. Check vxr simulation status (make sure it is in ‘running’ state)
root@r01:/# cd /nobackup/
root@r01:/nobackup# vxr.py status
{"localhost": "running"}

3. If simulation not in 'running' state, invoke sim-check command
root@r01:/nobackup# vxr.py sim-check

Try booting image outside of Containerlab
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
docker run --rm -it --privileged --env CLAB_INTFS=0 --name test 8201clab:7.5.3
