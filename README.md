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
docker load < 8201clab:7.5.3.tar.gz
```
3. Create a simple Cisco 8000 back-to-back clab topology file


cat example.yml
```
name: c8201

topology:
  kinds:
    linux:
      image: 8201clab:7.5.3
  nodes:
    r1:
      kind: linux
      binds: [r1.cfg:/startup.cfg:ro]
    r2:
      kind: linux
      binds: [r2.cfg:/startup.cfg:ro]

  links:
    - endpoints: ["r1:FH0_0_0_0", "r2:FH0_0_0_0"]
    - endpoints: ["r1:FH0_0_0_1", "r2:FH0_0_0_1"]
```
#### NOTE: Data interfaces naming convention 
FH0_0_0_[0..N] ->  400G interfaces

Hu0_0_0_[0..N] ->  100G interfaces

4. Create a simple r1 and r2 config for ssh and basic ping testing.

cat r1.cfg
```
hostname r1
interface FourHundredGigE 0/0/0/0
   ipv4 addr 10.0.0.1/24
   no shutdown
!
ssh server vrf default
```
cat r2.cfg
```
hostname r2
interface FourHundredGigE 0/0/0/0
   ipv4 addr 10.0.0.2/24
   no shutdown
!
ssh server vrf default
```

5. Deploy topology
```
containerlab deploy -t example.yml

INFO[0000] Creating container: "r1"
INFO[0000] Creating container: "r2"
INFO[0003] Creating virtual wire: r1:FH0_0_0_1 <--> r2:FH0_0_0_1 
INFO[0003] Creating virtual wire: r1:FH0_0_0_0 <--> r2:FH0_0_0_0 


+---+---------------+--------------+----------------+-------+---------+----------------+----------------------+
| # |     Name      | Container ID |     Image      | Kind  |  State  |  IPv4 Address  |     IPv6 Address     |
+---+---------------+--------------+----------------+-------+---------+----------------+----------------------+
| 1 | clab-c8201-r1 | 986c48fa97fc | 8201clab:7.5.3 | linux | running | 172.20.20.3/24 | 2001:172:20:20::3/64 |
| 2 | clab-c8201-r2 | 15a4932dcb20 | 8201clab:7.5.3 | linux | running | 172.20.20.2/24 | 2001:172:20:20::2/64 |
+---+---------------+--------------+----------------+-------+---------+----------------+----------------------+
```
NOTE: it takes about 6 minutes for XR to come up.


6. Monitor sim bringup
```
docker logs -f clab-c8201-r1
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
containerlab inspect -t example.yml |grep r1

| 1 | clab-c8201-r1 | de1b6f5a334b | 8201clab:7.5.3 | linux | running | 172.20.20.3/24 | 2001:172:20:20::2/64 |

ssh cisco@172.20.20.3
Password:


RP/0/RP0/CPU0:r1#show platform 
Thu Dec  1 23:30:09.277 UTC
Node              Type                     State                    Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        8201-32FH(Active)        IOS XR RUN               NSHUT
0/PM0             PSU2KW-ACPI              OPERATIONAL              NSHUT
0/PM1             PSU2KW-ACPI              OPERATIONAL              NSHUT
0/FT0             FAN-1RU-PI-V2            OPERATIONAL              NSHUT
...


RP/0/RP0/CPU0:r1#show version 
Thu Dec  1 23:30:18.434 UTC
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
r1 uptime is 12 minutes
Cisco 8201-32FH System
```

8. Run ping test.
```
RP/0/RP0/CPU0:r1#show ipv4 int br  FourHundredGigE 0/0/0/0
Interface                      IP-Address      Status                Protocol
FourHundredGigE0/0/0/0         10.0.0.1        Up                    Up

RP/0/RP0/CPU0:r1#ping 10.0.0.2
Mon Oct 10 17:05:33.104 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/15/47 ms
```

XR serial console access
========================


1. Telnet to serial console (port 60000)
```
docker exec -ti clab-c8201-r1 telnet 0 60000
Trying 0.0.0.0...
Connected to 0.
Escape character is '^]'.

RP/0/RP0/CPU0:r1#
```

Troubleshooting
===============


1. Invoke bash shell of the relevant docker instance
docker exec -ti clab-c8201-r1 bash
root@r1:/#

2. Check vxr simulation status (make sure it is in ‘running’ state)
root@r1:/# cd /nobackup/
root@r1:/nobackup# vxr.py status
{"localhost": "running"}

3. If simulation not in 'running' state, invoke sim-check command
root@r1:/nobackup# vxr.py sim-check

Try booting image outside of Containerlab
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
docker run --rm -it --privileged --env CLAB_INTFS=0 --name test 8201clab:7.5.3
