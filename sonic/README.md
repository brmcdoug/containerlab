Installing and running Cisco 8000 SONIC Containerlab docker image
=================================================================

#### Many thanks to Rafal Skorka for providing the 8000 image and original instructions

1. Host server requirements

   - verify that Open vSwitch is installed
     ovs-vsctl --version

   - limit /proc/sys/kernel/pid_max to 1048575
     sysctl -w kernel.pid_max=1048575

   - make sure KVM modules are installed and configured
     file /dev/kvm (the output should be /dev/kvm: character special)

   - cpu and memory: recommend 20GB and 4 cores per router

2. Install Containerlab:
```
https://containerlab.dev/install/
```

3. Install 8000 SONIC docker image
```   
docker load < c8000-clab-sonic:19.tar.gz
```

1. Copy sonic-cisco-8000.bin image to local storage (e.g. /sonic_images). Example:
```
ls /opt/images/ | grep sonic

c8000-clab-sonic:19.tar.gz
sonic-cisco-8000.bin
sonic-README
sonic.tar
```

4. Create a simple SONIC back-to-back clab topology file

    - NOTE: we will be using 'linux' ContainerLab kind to boot sonic images

cat example.yml
```
name: sonic

topology:
  kinds:
    linux:
        image: c8000-clab-sonic:19
        binds: 
            - /opt/images:/images
        env:
            IMAGE: /images/sonic-cisco-8000.bin
            PID: '8101-32H'
  nodes:
    r1:
      kind: linux
    r2:
      kind: linux

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
    - endpoints: ["r1:eth2", "r2:eth2"]
```

#### NOTE: Data interfaces naming convention 
```
eth1 -> first front panel port  (Ethernet0)
eth2 -> second front panel port (Ethernet4 or Ethernet8)
```

5. Deploy topology
```
sudo containerlab deploy -t example.yml
```
   - NOTE: it may 10 or more minutes for SONIC to come up

6. Monitor sonic device bringup
```
docker logs -f clab-sonic-r1 
```
   - After a while, you should see:
```
19:55:56 INFO Sim up
Router up
```
#### If you don't see "Router up" after 10 minutes, please share the "docker logs ..." output with Cisco team.

7. Test ssh to XR (use cisco/cisco123 login credentials)
```
sudo containerlab inspect -t example.yml |grep r1
```
```
| 1 | clab-sonic-r1 | 1edc2ce54d66 | c8000-clab-sonic:16 | linux | running | 172.20.20.3/24 | 2001:172:20:20::2/64 |
```
```
ssh cisco@172.20.20.3
cisco@172.20.20.3's password: 

Linux sonic 5.10.0-18-2-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64
You are on
  ____   ___  _   _ _  ____
 / ___| / _ \| \ | (_)/ ___|
 \___ \| | | |  \| | | |
  ___) | |_| | |\  | | |___
 |____/ \___/|_| \_|_|\____|

-- Software for Open Networking in the Cloud --

Last login: Tue Feb 21 19:56:42 2023 from 172.20.20.1
cisco@sonic:~$ 
```
8. Destroy topology
```
sudo containerlab destroy -t example.yml 
```

### SONIC serial console access

1. Telnet to serial console (port 60000)
```
docker exec -ti clab-sonic-r1 telnet 0 60000

Trying 0.0.0.0...
Connected to 0.
Escape character is '^]'.

sonic login: 
```
### Troubleshooting

1. Invoke bash shell of the relevant docker instance
```
docker exec -ti clab-sonic-r1 bash
root@r1:/#
```
2. Check vxr simulation status (make sure it is in ‘running’ state)
```
root@r1:/# cd /nobackup/
root@r1:/nobackup# vxr.py status
{"localhost": "running"}
```
3. If simulation not in 'running' state, invoke sim-check command
```
root@r1:/nobackup# vxr.py sim-check
```