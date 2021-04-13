---
layout: default
title: OpenVPN Setup (by Nao Devils Dortmund)
---

This guide describes briefly our VPN server setup used at vRoDEO2021. The main goal of our setup is to allow easy access to the robot's network in order to communicate as seamlessly as possible. In particular, it forwards all broadcasts (such as team communication or game controller messages) in both directions and it does not require any special configuration on the robots.

It is not meant for a "production setup" with a large number of connected clients or actual games that should not be interfered by other people. The setup is designed to behave like a physical connection to the local Ethernet switch at the field side in official SPL setups.

Therefore, we use [OpenVPN](https://openvpn.net/community-downloads/) in bridge mode (instead of the usually more common routed mode), which forwards all layer 2 Ethernet frames. If you are interested in a routed setup that has some other design goals, you can ask the HULKs for details.

## Server side

We will not go into every detail and list every command since there are many OpenVPN tutorials that cover all the configuration steps for different purposes. Instead, we will outline the basic steps and config files that are required for such a setup and assume some basic Linux and networking knowledge. Some tutorials that can be used for reference:

* [Very detailed tutorial for a routed OpenVPN setup on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-ubuntu-20-04)
* [Old tutorial for a bridged OpenVPN setup](https://help.ubuntu.com/community/OpenVPN)

### Prerequisites

* Linux-based operating system (e.g., Ubuntu/Debian)
* OpenVPN 2.4 (for Ubuntu, an official [OpenVPN PPA](https://community.openvpn.net/openvpn/wiki/OpenvpnSoftwareRepos) is recommended)
* OpenSSL (for certificate management)

### Network setup

The detailed configuration depends on the physical network setup present at your location and may be different from ours. Currently, we use three IP subnets that are bridged on the same network adapter:

* A public IP range by the university (e.g., 1.2.3.0/24),
* an IP range (10.0.0.0/16) used for the robot's WiFi, and
* a IP range (10.1.0.0/16) used for the robot's wired Ethernet connection.

However, it is sensible (and also planned for the future) to physically separate the public subnet from the robots using an additional network interface. In the past, this was not possible for use due to network constraints.

#### Bridge setup

First, you have to add and configure a bridge interface that connects the physical Ethernet connection (e.g., eth0) connected to the robots to the virtual OpenVPN tap network interface later. The way to do this depends on your Linux distribution and network manager.

For a command line configuration, the traditional way to do this is by using `brctl` (e.g., from the bridge-utlis package in Ubuntu) or more modern, this can also be achieved using `ip link`. The basic commands to handle this are outlined [here](https://wiki.archlinux.org/index.php/Network_bridge).

For a permanent configuration, distributions using the `/etc/network/interfaces` file can be configured as follows:

```
auto lo br0
iface lo inet loopback

iface br0 inet static
  address 1.2.3.4 # public IP address
  netmask 255.255.255.0
  gateway 1.2.3.1
  bridge_ports eth0

allow-hotplug eth0
iface eth0 inet manual
  up ip link set $IFACE up promisc on
  down ip link set $IFACE down promisc off
```

Newer versions of Ubuntu using netplan can be configured [similarly](https://netplan.io/examples/#configuring-network-bridges).

In contrast to the configuration above, if you decide to separate the public network (e.g., on eth1) from the robot network (on eth0), you can specify an IP address in the 10.0.0.0/16 range for br0.

### Key setup

OpenVPN usually authenticates clients using a public key infrastructure (PKI). Therefore, an own certificate authority that issues certificates to individual clients is required. We recommend the use of [easyrsa3](https://github.com/OpenVPN/easy-rsa) since it reduces the complexity to manage such an CA significantly.

We clone the easyrsa3 scripts into the `/etc/openvpn/easy-rsa` directory and store the generated PKI in `/etc/openvpn/pki`. Furthermore, we copy the example parameters `/etc/openvpn/easy-rsa/easyrsa3/vars.example` to a new `vars` file and set the following parameters:

```
set_var EASYRSA_PKI		"/etc/openvpn/pki"
set_var EASYRSA_ALGO		ec
set_var EASYRSA_CURVE		secp521r1
set_var EASYRSA_CERT_EXPIRE	3650
set_var EASYRSA_CRL_DAYS	3650
```

Details on these parameters are explained in the example file.

After that, change to the `/etc/openvpn/easy-rsa/easyrsa3` directory and execute the following commands to initialize your PKI and generate the certificates:

```
# Initializes the pki directory.
./easyrsa init-pki

# Generates a new certificates authority that will be used to issue the
# client and server certificates. "Common Name" can be set to whatever
# you want (e.g., your name). Remember the given password!
./easyrsa build-ca

# Generates an (at first empty) certificate revocation list that will be
# checked by OpenVPN whenever a client tries to connect.
./easyrsa gen-crl

# Generates a server certificate for OpenVPN with no password
# protection. Pass, e.g., the hostname or IP address you want to use as
# second parameter and enter the CA's password when requested.
./easyrsa build-server-full myserver.domain.com nopass

# Generates the first client certificate. Give it any name you want. You
# can also specify "nopass" as third parameter to disable encryption of
# the generated private key. Repeat this step for every client.
./easyrsa build-client-full myclient

# Generates a static key for TLS handshake encryption (better security)
openvpn --genkey --secret /etc/openvpn/pki/tc.key
```

### Server setup

Create a new OpenVPN configuration file in `/etc/openvpn/server.conf`. Details on all options can be found in the [official documentation](https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage). Our server is configured as follows:

```
# Use UDP protocol on port 1194
port 1194
proto udp6

# Enable tap interface and execute init script
dev tap0
script-security 2
up "/etc/openvpn/up.sh br0 tap0 1500"

# Specify certs and keys
ca ./pki/ca.crt
cert ./pki/issued/myserver.domain.com.crt
key ./pki/private/myserver.domain.com.key
crl-verify ./pki/crl.pem
tls-crypt ./pki/tc.key
dh none

# Enable server mode and specify IP range
mode server
ifconfig-pool 10.0.0.2 10.0.0.255 255.255.0.0

# Enable client-to-client communication
client-to-client

# Allow multiple clients to use the same certificate
duplicate-cn

# Send keep alive message every 60 seconds and timeout after 300 seconds
keepalive 60 300

# Some TLS and encryption options
cipher AES-256-GCM
auth SHA512
tls-server
tls-cipher TLS-ECDHE-ECDSA-WITH-AES-256-GCM-SHA384
tls-version-min 1.2
remote-cert-tls client

# Drop privileges
user nobody
group nogroup
persist-key
persist-tun

# Output status information
status openvpn-status.log

# Notify clients on server shutdown
explicit-exit-notify 1
```

For the initialization of the tap0 interface, we use the following small shell script in `/etc/openvpn/up.sh`, which enables promiscuous mode and connects to the given bridge interface. (Don't forget `chmod 775`.)

```
#!/bin/sh

BR="$1"
DEV="$2"
MTU="$3"
/sbin/ip link set "$DEV" up promisc on mtu "$MTU"
/sbin/ip link set "$DEV" master "$BR"
```

After that, you should be able to start the server, e.g., using `systemctl start openvpn`. If your location uses a firewall, make sure to forward UDP port 1194.

## Client side

For each client that wants to connect to your server, you need to provide an OpenVPN profile. We will provide a client config `myclient.opn` as a template, which can be adjusted to your needs:

```
# Use tap interface
dev tap

# Enter your server's IP address or hostname here
remote 1.2.3.4 1194 udp

# Client mode
client
nobind
persist-key
persist-tun

# Hide some warnings on WiFi networks
mute-replay-warnings

<ca>
-----BEGIN CERTIFICATE-----
[[ copy /etc/openvpn/pki/ca.crt here ]]
-----END CERTIFICATE-----
</ca>
<cert>
-----BEGIN CERTIFICATE-----
[[ copy /etc/openvpn/pki/issued/myclient.crt here ]]
-----END CERTIFICATE-----
</cert>
<key>
-----BEGIN PRIVATE KEY-----
[[ copy /etc/openvpn/pki/private/myclient.key here ]]
-----END PRIVATE KEY-----
</key>
<tls-crypt>
-----BEGIN OpenVPN Static key V1-----
[[ copy /etc/openvpn/pki/tc.key here ]]
-----END OpenVPN Static key V1-----
</tls-crypt>

# Some TLS and encryption options
cipher AES-256-GCM
auth SHA512
tls-client
tls-cipher TLS-ECDHE-ECDSA-WITH-AES-256-GCM-SHA384
tls-version-min 1.2
remote-cert-tls server

# Enable verbose mode
verb 3

# Notify server on disconnect
explicit-exit-notify 1
```

**For the certificates and keys, only copy the base64 encoded parts including the BEGIN and END markers into the profile.**

After that, you can import the profile into your OpenVPN client. On Windows, this can be done using the [official client from the website](https://openvpn.net/community-downloads/) from the menu in the system tray. On other platforms, this can usually be done using the graphical network manager or the command line.

After connecting successfully, you should be able to reach the remote 10.0.0.0/16 subnet. Since it is a bridged configuration, you can also add additional IP address, e.g., for the wired Ethernet subnet 10.1.0.0/16 or whatever you want, to your OpenVPN network interface if required. Sadly, OpenVPN does not support the automatic assignment of multiple IP address during the connection phase.

## Questions and feedback?

If anything does not work as expected or if there are ideas for further improvement of this guide, please tell us via mail (aaron.larisch@tu-dortmund.de) or contact us on Discord.
