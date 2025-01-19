# Lab 7: VPN Lab: The Container Version

```
Copyright © 2020 by Wenliang Du.
This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International
License. If you remix, transform, or build upon the material, this copyright notice must be left intact, or
reproduced in a way that is reasonable to the medium in which the work is being re-published.
```
## 1 Overview

A Virtual Private Network (VPN) is a private network built on top of a public network, usually the Internet.
Computers inside a VPN can communicate securely, just like if they were on a real private network that is
physically isolated from outside, even though their traffic may go through a public network. VPN enables
employees to securely access a company’s intranet while traveling; it also allows companies to expand their
private networks to places across the country and around the world.
The objective of this lab is to help students understand how VPN works. We focus on a specific type of
VPN (the most common type), which is built on top of the transport layer. We will build a very simple VPN
from the scratch, and use the process to illustrate how each piece of the VPN technology works. A real VPN
program has two essential pieces, tunneling and encryption. This lab only focuses on the tunneling part,
helping students understand the tunneling technology, so the tunnel in this lab is not encrypted. The lab covers the following topics:

- Virtual Private Network
- The TUN/TAP virtual interface
- IP tunneling
- Routing

**Lab environment**. Files needed for this lab are included in Labsetup.zip, which can be fetched by running the following commands.

```
# Download the lab setup files
$ sudo wget https://seedsecuritylabs.org/Labs_20.04/Files/VPN_Tunnel/Labsetup.zip

# Unzip the lab setup files
$ sudo unzip Labsetup.zip
```

## 2 Task 1: Network Setup

We will create a VPN tunnel between a computer (client) and a gateway, allowing the computer to securely
access a private network via the gateway. We need at least three machines: VPN client (also serving as
HostU), VPN server (the router/gateway), and a host in the private network (HostV). The network setup is
depicted in Figure 1.

![image](https://github.com/CloudLabs-MOC/CloudLabs-SEED/assets/33658792/35c265b6-8081-40b8-b9c7-bd580d817ad3)

`Figure 1: Lab environment setup`

In practice, the VPN client and VPN server are connected via the Internet. For the sake of simplicity,
we directly connect these two machines to the same LAN in this lab, i.e., this LAN simulates the Internet.
The third machine, HostV, is a computer inside the private network. Users on HostU (outside of the
private network) want to communicate with HostV via the VPN tunnel. To simulate this setup, we connect
HostV to VPN Server (also serving as a gateway). In such a setup, HostV is not directly accessible from
the Internet; nor is it directly accessible from HostU.

**Lab setup**. Please download the Labsetup.zip file to your VM from the lab’s website, unzip it, enter
the Labsetup folder, and use the docker-compose.yml file to set up the lab environment. Detailed
explanation of the content in this file and all the involved `Dockerfile` can be found from the user manual,
which is linked to the website of this lab. 

In the following, we list some of the commonly used commands related to Docker and Compose. Since
we are going to use these commands very frequently, we have created aliases for them in the .bashrc file
(in our provided SEEDUbuntu 20.04 VM).

**Build the Docker Container**. Use Docker Compose to build the container image. This step prepares the environment for running your web server with the required configurations.

```
# Build the Docker container
$ docker-compose build

# OR use the alias
$ dcbuild
```

**Start the Docker Container**. This command initializes and runs the container based on the configurations specified in the docker-compose.yml file.

```
# Start the Docker container
$ docker-compose up

# OR use the alias
$ dcup
```

**Stop and Shut Down the Docker Container**. When you’re finished or need to reset the environment, shut down the running container to release resources.

```
# Stop and shut down the Docker container
$ docker-compose down

# OR use the alias
$ dcdown
```
 
All the containers will be running in the background. To run commands on a container, we often need
to get a shell on that container. We first need to use the `"docker ps"` command to find out the ID of
the container, and then use `"docker exec"` to start a shell on that container. We have created aliases for
them in the `.bashrc` file.

**List Running Docker Containers**. Use the alias dockps to view a list of running containers, displaying each container's ID and name in a simplified format.

```
$ dockps
```

#### Output
The output will list all running Docker containers, each with its unique ID and assigned name. An example output might look like this:

```
b1004832e275 hostA-10.9.0.5
0af4ea7a3e2e hostB-10.9.0.6
9652715c8e0a hostC-10.9.0.7
```
Each line includes the container ID and its corresponding name (in this case, hostA, hostB, and hostC), along with their assigned IP addresses.

**Access a Specific Container’s Shell**. To open a shell inside a specific container, use the alias docksh followed by the first few characters of the container's ID. For example, to access hostC, use the ID prefix 96 (from the third line in the previous output).

```
$ docksh 96
```

#### Output
You will be logged into a shell session inside the specified container. The prompt will change to show the container ID, indicating that you are now inside the container:

```
root@9652715c8e0a:/#
```

#### Note: If a Docker command requires the container ID, you only need to type the first few characters, as long as they are unique among all running containers.


**Shared folder**. In this lab, we need to write our own code and run it inside containers. Code editing is
more convenient inside the VM than in containers, because we can use our favorite editors. In order for the
VM and container to share files, we have created a shared folder between the VM and the container using
the Docker volumes. If you look at the Docker Compose file, you will find out that we have added the
following entry to some of the containers. It indicates mounting the ./volumes folder on the host machine
(i.e., the VM) to the /volumes folder inside the container. We will write our code in the./volumes
folder (on the VM), so they can be used inside the containers.

```
volumes:
    - ./volumes:/volumes
```

**Packet sniffing**. Being able to sniffing packets is very important in this lab, because if things do not go
as expected, being able to look at where packets go can help us identify the problems. There are several
different ways to do packet sniffing:

- Running `tcpdump` on containers. We have already installed `tcpdump` on each container. To sniff
    the packets going through a particular interface, we just need to find out the interface name, and then
    do the following (assume that the interface name is eth0):

  ```
    # tcpdump -i eth0 -n
  ```
  ![tun](images/lab7-888.png)

  
It should be noted that inside containers, due to the isolation created by Docker, when we run `tcpdump` inside a container, we can only sniff the packets going in and out of this container. We won’t be able to sniff the packets between other containers. However, if a container uses the host mode in its network setup, it can sniff other containers’ packets.

- Running `tcpdump` on the VM. If we run `tcpdump` on the VM, we do not have the restriction on the
    containers, and we can sniff all the packets going among containers. The interface name for a network
    is different on the VM than on the container. On containers, each interface name usually starts with
    eth; on the VM, the interface name for the network created by Docker starts with br-, followed by
    the ID of the network. You can always use the `ip address` command to get the interface name on
    the VM and containers.
  
   ![tun](images/lab7-777.png)

  - We can also run Wireshark on the VM to sniff packets. Similar to `tcpdump`, we need to select what
    interface we want Wireshark to sniff on.

**Testing**. Please conduct the following testings to ensure that the lab environment is set up correctly:

- HostU can communicate with VPN Server.
- VPN Server can communicate with HostV.
- HostU should not be able to communicate with HostV.
- Run tcpdump on the router, and sniff the traffic on each of the network. Show that you can capture
    packets.

     - From HostU (Client), ping the VPN  Server to confirm communication using `ping 10.9.0.11`
     
        ![tun](images/lab7-111.png)
       
     - From HostU (Client), attempt to ping HostV using `ping 192.168.60.5` to ensure it is not accessible.
     
        ![tun](images/lab7-222.png)
     
     - From VPN Server (Router), ping HostU to verify connectivity using `ping 10.9.0.5`.
     
        ![tun](images/lab7-333.png)
       
     - From VPN Server (Router), ping HostV to confirm access to the private network using `ping 192.168.60.5`.
     
        ![tun](images/lab7-444.png)

## 3 Task 2: Create and Configure TUN Interface

The VPN tunnel that we are going to build is based on the TUN/TAP technologies. TUN and TAP are virtual
network kernel drivers; they implement network device that are supported entirely in software. TAP (as in
network tap) simulates an Ethernet device and it operates with layer-2 packets such as Ethernet frames;
TUN (as in network TUNnel) simulates a network layer device and it operates with layer-3 packets such as
IP packets. With TUN/TAP, we can create virtual network interfaces.
A user-space program is usually attached to the TUN/TAP virtual network interface. Packets sent by an
operating system via a TUN/TAP network interface are delivered to the user-space program. On the other
hand, packets sent by the program via a TUN/TAP network interface are injected into the operating system
network stack. To the operating system, it appears that the packets come from an external source through
the virtual network interface.
When a program is attached to a TUN/TAP interface, IP packets sent by the kernel to this interface will
be piped into the program. On the other hand, IP packets written to the interface by the program will be
piped into the kernel, as if they came from the outside through this virtual network interface. The program
can use the standard read() and write() system calls to receive packets from or send packets to the
virtual interface.
The objective of this task is to get familiar with the TUN/TAP technology. We will conduct several
experiments to learn the technical details of the TUN/TAP interface. We will use the following Python
program as the basis for the experiments, and we will modify this base code throughout this lab. The code
is already included in the `volumes` folder in the zip file.

`Listing 1: Creating a TUN interface (tun.py)`

```
#!/usr/bin/env python3

import fcntl
import struct
import os
import time
from scapy.all import *

TUNSETIFF = 0x400454ca
IFF_TUN   = 0x0001
IFF_TAP   = 0x0002
IFF_NO_PI = 0x1000

# Create the tun interface
tun = os.open("/dev/net/tun", os.O_RDWR)
ifr = struct.pack('16sH', b'tun%d', IFF_TUN | IFF_NO_PI)
ifname_bytes  = fcntl.ioctl(tun, TUNSETIFF, ifr)

# Get the interface name
ifname = ifname_bytes.decode('UTF-8')[:16].strip("\x00")
print("Interface Name: {}".format(ifname))

while True:
   time.sleep(10)
```

### 3.1 Task 2.a: Name of the Interface

We will run the `tun.py` program on HostU. Make the `tun.py` program executable and run it using root privileges. 
To modify and prepare `tun.py`, update the program to set a custom prefix for the interface name, as shown in the following figure. You can access the Python file by navigate to the `tun.py` file in the shared directory `volume`, as shown in the figure.

  ![tun](images/lab7-555.png)

You need to update the following line in the program `ifr = struct.pack('16sH', b'tun%d', IFF_TUN | IFF_NO_PI)`.

  ![tun](images/lab7-666.png)

```
// Make the Python program executable
# chmod a+x tun.py

// Run the program using the root privilege
# tun.py
```

Once the program is executed, it will block. You can go to another terminal and get a new shell on the
container. Then print out all the interfaces on the machine. Please report your observation after running the
following command:

```
# ip address
```
  ![tun](images/lab7-999.png)

You should be able to find an interface called `tun0`. Your job in this task is to change the `tun.py`
program, so instead of using `tun` as the prefix of the interface name, use your last name as the prefix. For
example, if your last name is smith, you should use smith as the prefix. If your last name is long, you can
use the first five characters. Please show your results.

### 3.2 Task 2.b: Set up the TUN Interface

At this point, the TUN interface is not usable, because it has not been configured yet. There are two things
that we need to do before the interface can be used. First, we need to assign an IP address to it. Second, we
need to bring up the interface, because the interface is still in the down state. We can use the following two
commands for the configuration:

```
// Assign IP address to the interface
# ip addr add 192.168.53.99/24 dev tun


// Bring up the interface
# ip link set dev tun0 up
```

To make life easier, students can add the following two lines of code totun.py, so the configuration
can be automatically performed by the program.

```
os.system("ip addr add 192.168.53.99/24 dev {}".format(ifname))
os.system("ip link set dev {} up".format(ifname))
```

The following is the updated code with crating the TUN interface:

```
#!/usr/bin/env python3

import fcntl
import struct
import os
import time
from scapy.all import *

TUNSETIFF = 0x400454ca
IFF_TUN   = 0x0001
IFF_TAP   = 0x0002
IFF_NO_PI = 0x1000

# Create the tun interface
tun = os.open("/dev/net/tun", os.O_RDWR)
ifr = struct.pack('16sH', b'tun%d', IFF_TUN | IFF_NO_PI)
ifname_bytes  = fcntl.ioctl(tun, TUNSETIFF, ifr)

# Get the interface name
ifname = ifname_bytes.decode('UTF-8')[:16].strip("\x00")
print("Interface Name: {}".format(ifname))

# Configure the TUN interface
os.system("ip addr add 192.168.53.99/24 dev {}".format(ifname))
os.system("ip link set dev {} up".format(ifname))

while True:
   time.sleep(10)
```
After running the two commands above and the `ip address` command again, the TUN interface changes from DOWN to UP, and it is assigned the IP address 192.168.53.99/24. This makes the interface active and ready to send or receive packets, enabling it to function as part of the VPN tunnel.

  ![tun](images/lab7-10.png)

### 3.3 Task 2.c: Read from the TUN Interface

In this task, we will read from the TUN interface. Whatever coming out from the TUN interface is an IP
packet. We can cast the data received from the interface into a ScapyIPobject, so we can print out each
field of the IP packet. Please use the followingwhileloop to replace the one intun.py:

```
while True:
    # Get a packet from the tun interface
    packet = os.read(tun, 2048)
    if packet:
        ip = IP(packet)
        print(ip.summary())
```

Please run the revisedtun.pyprogram on HostU, configure the TUN interface accordingly, and then
conduct the following experiments. Please describe your observations:

- On HostU,pinga host in the 192.168.53.0/24 network. What are printed out by thetun.py
    program? What has happened? Why?
- On HostU,pinga host in the internal network 192.168.60.0/24, Doestun.pyprint out
    anything? Why?

### 3.4 Task 2.d: Write to the TUN Interface

In this task, we will write to the TUN interface. Since this is a virtual network interface, whatever is written
to the interface by the application will appear in the kernel as an IP packet.
We will modify thetun.pyprogram, so after getting a packet from the TUN interface, we construct a
new packet based on the received packet. We then write the new packet to the TUN interface. How the new
packet is constructed is up to students. The code in the following shows an example of how to write an IP
packet to the TUN interface.

```
# Send out a spoof packet using the tun interface
newip = IP(src=’1.2.3.4’, dst=ip.src)
newpkt = newip/ip.payload
os.write(tun, bytes(newpkt))
```

Please modify thetun.pycode according to the following requirements:

- After getting a packet from the TUN interface, if this packet is an ICMP echo request packet, construct
    a corresponding echo reply packet and write it to the TUN interface. Please provide evidence to show
    that the code works as expected.
- Instead of writing an IP packet to the interface, write some arbitrary data to the interface, and report
    your observation.

## 4 Task 3: Send the IP Packet to VPN Server Through a Tunnel

In this task, we will put the IP packet received from the TUN interface into the UDP payload field of a new
IP packet, and send it to another computer. Namely, we place the original packet inside a new packet. This
is called IP tunneling. The tunnel implementation is just standard client/server programming. It can be built
on top of TCP or UDP. In this task, we will use UDP. Namely, we put an IP packet inside the payload field
of a UDP packet.

The server program **tunserver.py**. We will runtunserver.pyprogram on VPN Server. This
program is just a standard UDP server program. It listens to port 9090 and print out whatever is received.
The program assumes that the data in the UDP payload field is an IP packet, so it casts the payload to a
ScapyIPobject, and print out the source and destination IP address of the enclosed IP packet.

`Listing 2:tunserver.py`

```
#!/usr/bin/env python

from scapy.all import *

IP_A = "0.0.0.0"
PORT = 9090

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind((IP_A, PORT))

while True:
    data, (ip, port) = sock.recvfrom(2048)
    print("{}:{} --> {}:{}".format(ip, port, IP_A, PORT))
    pkt = IP(data)
    print(" Inside: {} --> {}".format(pkt.src, pkt.dst))
```

Implement the client program **tunclient.py**. First, we need to modify the TUN programtun.py.
Let’s rename it, and call ittunclient.py. Sending data to another computer using UDP can be done
using the standard socket programming.
Replace thewhileloop in the program with the following: The SERVERIP an dSERVERPORT should be replaced with the actual IP address and port number of the server program running on VPN
Server.

```
# Create UDP socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

while True:
    # Get a packet from the tun interface
    packet = os.read(tun, 2048)
    if packet:
        # Send the packet via the tunnel
        sock.sendto(packet, (SERVER_IP, SERVER_PORT))
```

Testing. Run thetunserver.pyprogram on VPN Server, and then runtunclient.pyon Host
U. To test whether the tunnel works or not,ping any IP address belonging to the 192.168.53.0/
network. What is printed out on VPN Server? Why?
Our ultimate goal is to access the hosts inside the private network 192.168.60.0/24 using the
tunnel. Let usping HostV, and see whether the ICMP packet is sent to VPN Server through the tunnel.
If not, what are the problems? You need to solve this problem, so the ping packet can be sent through
the tunnel. This is done through routing, i.e., packets going to the 192.168.60.0/24 network should
be routed to the TUN interface and be given to thetunclient.pyprogram. The following command
shows how to add an entry to the routing table:

```
# ip route add <network> dev <interface> via <router ip>
```

Please demonstrate that when you ping an IP address in the192.168.60.0/
network, the ICMP packets are received bytunserver.pythrough the tunnel.

## 5 Task 4: Set Up the VPN Server

Aftertunserver.pygets a packet from the tunnel, it needs to feed the packet to the kernel, so the kernel
can route the packet towards its final destination. This needs to be done through a TUN interface, just like
what we did in Task 2. Please modifytunserver.py, so it can do the following:

- Create a TUN interface and configure it.
- Get the data from the socket interface; treat the received data as an IP packet.
- Write the packet to the TUN interface.

Before running the modifiedtunserver.py, we need to enable the IP forwarding. Unless specifi-
cally configured, a computer will only act as a host, not as a gateway. VPN Server needs to forward packets
between the private network and the tunnel, so it needs to function as a gateway. We need to enable the IP
forwarding for a computer to behave like a gateway. IP forwarding has already been enabled on the router
container. You can see indocker-compose.ymlthat the router container has the following entry:

```
sysctls:
    - net.ipv4.ip_forward=1
```

Testing. If everything is set up properly, we canpingHostVfrom HostU. The ICMP echo request
packets should eventually arrive at HostVthrough the tunnel. Please show your proof. It should be noted
that although HostVwill respond to the ICMP packets, the reply will not get back to HostU, because we
have not set up everything yet. Therefore, for this task, it is sufficient to show (using Wireshark or tcpdump)
that the ICMP packets have arrived at HostV.


## 6 Task 5: Handling Traffic in Both Directions

After getting to this point, one direction of your tunnel is complete, i.e., we can send packets from HostU
to HostVvia the tunnel. If we look at the Wireshark trace on HostV, we can see that HostVhas sent out
the response, but the packet gets dropped somewhere. This is because our tunnel is only one directional; we
need to set up its other direction, so returning traffic can be tunneled back to Host U.
To achieve that, our TUN client and server programs need to read data from two interfaces, the TUN
interface and the socket interface. All these interfaces are represented by file descriptors, so we need to
monitor them to see whether there are data coming from them. One way to do that is to keep polling them,
and see whether there are data on each of the interfaces. The performance of this approach is undesirable,
because the process has to keep running in an idle loop when there is no data. Another way is to read from
an interface. By default, read is blocking, i.e., the process will be suspended if there are no data. When
data become available, the process will be unblocked, and its execution will continue. This way, it does not
waste CPU time when there is no data.
The read-based blocking mechanism works well for one interface. If a process is waiting on multiple
interfaces, it cannot block on just one of the interfaces. It has to block on all of them altogether.Linuxhas a
system call calledselect(), which allows a program to monitor multiple file descriptors simultaneously.
To useselect(), we need to store all the file descriptors to be monitored in a set, and then we give the
set to theselect()system call, which will block the process until data are available on one of the file
descriptors in the set. We can check which file descriptor has received data. In the following Python code
snippet, we useselect()to monitor aTUNand a socket file descriptor.

```
# We assume that sock and tun file descriptors have already been created.

while True:
    # this will block until at least one interface is ready
    ready, _, _ = select.select([sock, tun], [], [])
    
    for fd in ready:
        if fd is sock:
            data, (ip, port) = sock.recvfrom(2048)
            pkt = IP(data)
            print("From socket <==: {} --> {}".format(pkt.src, pkt.dst))
            ... (code needs to be added by students) ...
    
    if fd is tun:
        packet = os.read(tun, 2048)
        pkt = IP(packet)
        print("From tun ==>: {} --> {}".format(pkt.src, pkt.dst))
        ... (code needs to be added by students) ...
```

Students can use the code above to replace thewhileloop in their TUN client and server programs.
The code is incomplete; students are expected to complete it.

Testing. Once this is done, we should be able to communicate with MachineV from MachineU, and the
VPN tunnel (un-encrypted) is now complete. Please show your wireshark proof using aboutpingand
telnet commands. In your proof, you need to point out how your packets flow.

### You have successfully completed the lab
