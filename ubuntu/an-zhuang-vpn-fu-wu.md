# 安装 VPN服务

## Introduction

Want to access the Internet safely and securely from your smartphone or laptop when connected to an untrusted network such as the WiFi of a hotel or coffee shop? A Virtual Private Network \(VPN\) allows you to traverse untrusted networks privately and securely as if you were on a private network. The traffic emerges from the VPN server and continues its journey to the destination.

When combined with HTTPS connections, this setup allows you to secure your wireless logins and transactions. You can circumvent geographical restrictions and censorship, and shield your location and any unencrypted HTTP traffic from the untrusted network.

OpenVPN is a full-featured open source Secure Socket Layer \(SSL\) VPN solution that accommodates a wide range of configurations. In this tutorial, we'll set up an OpenVPN server on a Droplet and then configure access to it from Windows, OS X, iOS and Android. This tutorial will keep the installation and configuration steps as simple as possible for these setups.

## Prerequisites

To complete this tutorial, you will need access to an Ubuntu 16.04 server.

You will need to configure a non-root user with sudo privileges before you start this guide. You can follow our Ubuntu 16.04 initial server setup guide to set up a user with appropriate permissions. The linked tutorial will also set up a firewall, which we will assume is in place during this guide.

When you are ready to begin, log into your Ubuntu server as your sudo user and continue below.

## Step 1: Install OpenVPN

To start off, we will install OpenVPN onto our server. OpenVPN is available in Ubuntu's default repositories, so we can use `apt` for the installation. We will also be installing the `easy-rsa` package, which will help us set up an internal CA \(certificate authority\) for use with our VPN.

To update your server's package index and install the necessary packages type:

```bash
sudo apt-get update
sudo apt-get install openvpn easy-rsa
```

The needed software is now on the server, ready to be configured.

## Step 2: Set Up the CA Directory

OpenVPN is an TLS/SSL VPN. This means that it utilizes certificates in order to encrypt traffic between the server and clients. In order to issue trusted certificates, we will need to set up our own simple certificate authority \(CA\).

To begin, we can copy the `easy-rsa` template directory into our home directory with the `make-cadir` command:

```bash
make-cadir ~/openvpn-ca
```

Move into the newly created directory to begin configuring the CA:

```bash
cd ~/openvpn-ca
```

## Step 3: Configure the CA Variables

To configure the values our CA will use, we need to edit the vars file within the directory. Open that file now in your text editor:

```bash
vi vars
```

Inside, you will find some variables that can be adjusted to determine how your certificates will be created. We only need to worry about a few of these.

Towards the bottom of the file, find the settings that set field defaults for new certificates. It should look something like this:

{% code-tabs %}
{% code-tabs-item title="~/openvpn-ca/vars" %}
```bash
. . .

export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="SanFrancisco"
export KEY_ORG="Fort-Funston"
export KEY_EMAIL="me@myhost.mydomain"
export KEY_OU="MyOrganizationalUnit"

. . .
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Edit the values in red to whatever you'd prefer, but do not leave them blank:

{% code-tabs %}
{% code-tabs-item title="~/openvpn-ca/vars" %}
```bash
. . .

export KEY_COUNTRY="US"
export KEY_PROVINCE="NY"
export KEY_CITY="New York City"
export KEY_ORG="DigitalOcean"
export KEY_EMAIL="admin@example.com"
export KEY_OU="Community"

. . .
```
{% endcode-tabs-item %}
{% endcode-tabs %}

While we are here, we will also edit the `KEY_NAME` value just below this section, which populates the subject field. To keep this simple, we'll call it `server` in this guide:

{% code-tabs %}
{% code-tabs-item title="~/openvpn-ca/vars" %}
```bash
export KEY_NAME="server"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

When you are finished, save and close the file.

## Step 4: Build the Certificate Authority

Now, we can use the variables we set and the `easy-rsa` utilities to build our certificate authority.

Ensure you are in your CA directory, and then source the `vars` file you just edited:

```bash
cd ~/openvpn-ca
source vars
```

You should see the following if it was sourced correctly:

```text
NOTE: If you run ./clean-all, I will be doing a rm -rf on /home/sammy/openvpn-ca/keys
```

Make sure we're operating in a clean environment by typing:

```bash
./clean-all
```

Now, we can build our root CA by typing:

```bash
./build-ca
```

This will initiate the process of creating the root certificate authority key and certificate. Since we filled out the vars file, all of the values should be populated automatically. Just press `ENTER` through the prompts to confirm the selections:

```text
Generating a 2048 bit RSA private key
..........................................................................................+++
...............................+++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [US]:
State or Province Name (full name) [NY]:
Locality Name (eg, city) [New York City]:
Organization Name (eg, company) [DigitalOcean]:
Organizational Unit Name (eg, section) [Community]:
Common Name (eg, your name or your server's hostname) [DigitalOcean CA]:
Name [server]:
Email Address [admin@email.com]:
```

We now have a CA that can be used to create the rest of the files we need.

## Step 5: Create the Server Certificate, Key, and Encryption Files

Next, we will generate our server certificate and key pair, as well as some additional files used during the encryption process.

Start by generating the OpenVPN server certificate and key pair. We can do this by typing:

{% hint style="warning" %}
If you choose a name other than `server` here, you will have to adjust some of the instructions below. For instance, when copying the generated files to the `/etc/openvpn` directroy, you will have to substitute the correct names. You will also have to modify the `/etc/openvpn/server.conf` file later to point to the correct `.crt` and `.key` files.
{% endhint %}

```bash
./build-key-server server
```

Once again, the prompts will have default values based on the argument we just passed in \(`server`\) and the contents of our `vars` file we sourced.

Feel free to accept the default values by pressing `ENTER`. Do not enter a challenge password for this setup. Towards the end, you will have to enter `y` to two questions to sign and commit the certificate:

```aspnet
. . .

Certificate is to be certified until May  1 17:51:16 2026 GMT (3650 days)
Sign the certificate? [y/n]: y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

Next, we'll generate a few other items. We can generate a strong Diffie-Hellman keys to use during key exchange by typing:

```bash
./build-dh
```

This might take a few minutes to complete.

Afterwards, we can generate an HMAC signature to strengthen the server's TLS integrity verification capabilities:

```bash
openvpn --genkey --secret keys/ta.key
```

## Step 6: Generate a Client Certificate and Key Pair

Next, we can generate a client certificate and key pair. Although this can be done on the client machine and then signed by the server/CA for security purposes, for this guide we will generate the signed key on the server for the sake of simplicity.

We will generate a single client key/certificate for this guide, but if you have more than one client, you can repeat this process as many times as you'd like. Pass in a unique value to the script for each client.

Because you may come back to this step at a later time, we'll re-source the `vars` file. We will use `client1` as the value for our first certificate/key pair for this guide.

To produce credentials without a password, to aid in automated connections, use the `build-key` command like this:

```bash
cd ~/openvpn-ca
source vars
./build-key-pass client1
```

Again, the defaults should be populated, so you can just hit `ENTER` to continue. Leave the challenge password blank and make sure to enter `y` for the prompts that ask whether to sign and commit the certificate.

## Configure the OpenVPN Service

Next, we can begin configuring the OpenVPN service using the credentials and files we've generated.

### Copy the Files to the OpenVPN Directory

To begin, we need to copy the files we need to the `/etc/openvpn` configuration directory.

We can start with all of the files that we just generated. These were placed within the `~/openvpn-ca/keys` directory as they were created. We need to move our CA cert, our server cert and key, the HMAC signature, and the Diffie-Hellman file:

```bash
cd ~/openvpn-ca/keys
sudo cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn
```

Next, we need to copy and unzip a sample OpenVPN configuration file into configuration directory so that we can use it as a basis for our setup:

```bash
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf
```

### Adjust the OpenVPN Configuration

Now that our files are in place, we can modify the server configuration file:

```bash
sudo vi /etc/openvpn/server.conf
```

Basic Configuration

First, find the HMAC section by looking for the `tls-auth` directive. Remove the ";" to uncomment the `tls-auth` line. Below this, add the `key-direction` parameter set to "0":

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
tls-auth ta.key 0 # This file is secret
key-direction 0
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Next, find the section on cryptographic ciphers by looking for the commented out `cipher` lines. The `AES-128-CBC` cipher offers a good level of encryption and is well supported. Remove the ";" to uncomment the cipher `AES-128-CBC` line:

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
cipher AES-128-CBC
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Below this, add an `auth` line to select the HMAC message digest algorithm. For this, `SHA256` is a good choice:

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
auth SHA256
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Finally, find the `user` and `group` settings and remove the ";" at the beginning of to uncomment those lines:

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
user nobody
group nogroup
```
{% endcode-tabs-item %}
{% endcode-tabs %}

\(Optional\) Push DNS Changes to Redirect All Traffic Through the VPN

The settings above will create the VPN connection between the two machines, but will not force any connections to use the tunnel. If you wish to use the VPN to route all of your traffic, you will likely want to push the DNS settings to the client computers.

You can do this, uncomment a few directives that will configure client machines to redirect all web traffic through the VPN. Find the `redirect-gateway` section and remove the semicolon ";" from the beginning of the `redirect-gateway` line to uncomment it:

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
push "redirect-gateway def1 bypass-dhcp"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

`push "redirect-gateway def1 bypass-dhcp"` 会路由所有流量到VPN服务器，而有时我们只需要访问VPN背后的局域网，那么我们可以改为：

```text
push "route 10.9.0.0 255.255.0.0"
```

`10.9.0.0 255.255.0.0`是VPN背后的局域网。

Just below this, find the `dhcp-option` section. Again, remove the ";" from in front of both of the lines to uncomment them:

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This should assist clients in reconfiguring their DNS settings to use the VPN tunnel for as the default gateway.

\(Optional\) Adjust the Port and Protocol

By default, the OpenVPN server uses port 1194 and the UDP protocol to accept client connections. If you need to use a different port because of restrictive network environments that your clients might be in, you can change the `port` option. If you are not hosting web content your OpenVPN server, port 443 is a popular choice since this is usually allowed through firewall rules.

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
# Optional!
port 443
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Often if the protocol will be restricted to that port as well. If so, change `proto` from `UDP` to `TCP`:

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
# Optional!
proto tcp
```
{% endcode-tabs-item %}
{% endcode-tabs %}

\(Optional\) Point to Non-Default Credentials

If you selected a different name during the `./build-key-server` command earlier, modify the cert and key lines that you see to point to the appropriate `.crt` and `.key` files. If you used the default server, this should already be set correctly:

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
cert server.crt
key server.key
```
{% endcode-tabs-item %}
{% endcode-tabs %}

When you are finished, save and close the file.

## Step 8: Adjust the Server Networking Configuration

Next, we need to adjust some aspects of the server's networking so that OpenVPN can correctly route traffic.

### Allow IP Forwarding

First, we need to allow the server to forward traffic. This is fairly essential to the functionality we want our VPN server to provide.

We can adjust this setting by modifying the `/etc/sysctl.conf` file:

{% code-tabs %}
{% code-tabs-item title="/etc/sysctl.conf" %}
```text
net.ipv4.ip_forward=1
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Save and close the file when you are finished.

To read the file and adjust the values for the current session, type:

```bash
sudo sysctl -p
```

### Adjust the UFW Rules to Masquerade Client Connections

If you followed the Ubuntu 16.04 initial server setup guide in the prerequisites, you should have the UFW firewall in place. Regardless of whether you use the firewall to block unwanted traffic \(which you almost always should do\), we need the firewall in this guide to manipulate some of the traffic coming into the server. We need to modify the rules file to set up masquerading, an `iptables` concept that provides on-the-fly dynamic NAT to correctly route client connections.

Before we open the firewall configuration file to add masquerading, we need to find the public network interface of our machine. To do this, type:

```bash
ip route
```

Your public interface should follow the word "`dev`". For example, this result shows the interface named `eth0`, which is highlighted below:

```text
default via 10.9.0.1 dev eth0 onlink
10.9.0.0/16 dev eth0  proto kernel  scope link  src 10.9.50.143
```

When you have the interface associated with your default route, open the `/etc/ufw/before.rules` file to add the relevant configuration:

```bash
sudo vi /etc/ufw/before.rules
```

This file handles configuration that should be put into place before the conventional UFW rules are loaded. Towards the top of the file, add the highlighted lines below. This will set the default policy for the `POSTROUTING` chain in the `nat` table and masquerade any traffic coming from the VPN:

{% hint style="warning" %}
Remember to replace `eth0` in the `-A POSTROUTING` line below with the interface you found in the above command.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="/etc/ufw/before.rules" %}
```text
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#

# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0] 
# Allow traffic from OpenVPN client to eth0 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES

# Don't delete these required lines, otherwise there will be errors
*filter
. . .
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Save and close the file when you are finished.

We need to tell UFW to allow forwarded packets by default as well. To do this, we will open the `/etc/default/ufw` file:

```bash
sudo vi /etc/default/ufw
```

Inside, find the `DEFAULT_FORWARD_POLICY` directive. We will change the value from `DROP` to `ACCEPT`:

{% code-tabs %}
{% code-tabs-item title="/etc/default/ufw" %}
```text
DEFAULT_FORWARD_POLICY="ACCEPT"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Save and close the file when you are finished.

### Open the OpenVPN Port and Enable the Changes

Next, we'll adjust the firewall itself to allow traffic to OpenVPN.

If you did not change the port and protocol in the `/etc/openvpn/server.conf` file, you will need to open up `UDP` traffic to port `1194`. If you modified the port and/or protocol, substitute the values you selected here.

We'll also add the SSH port in case you forgot to add it when following the prerequisite tutorial:

```bash
sudo ufw allow 1194/udp
sudo ufw allow OpenSSH
```

Now, we can disable and re-enable UFW to load the changes from all of the files we've modified:

```bash
sudo ufw disable
sudo ufw enable
```

Our server is now configured to correctly handle OpenVPN traffic.

## Step 9: Start and Enable the OpenVPN Service

We're finally ready to start the OpenVPN service on our server. We can do this using systemd.

We need to start the OpenVPN server by specifying our configuration file name as an instance variable after the systemd unit file name. Our configuration file for our server is called `/etc/openvpn/server.conf`, so we will add `@server` to end of our unit file when calling it:

```bash
sudo systemctl start openvpn@server
```

Double-check that the service has started successfully by typing:

```bash
sudo systemctl status openvpn@server
```

If everything went well, your output should look something that looks like this:

```text
openvpn@server.service - OpenVPN connection to server
   Loaded: loaded (/lib/systemd/system/openvpn@.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2016-05-03 15:30:05 EDT; 47s ago
     Docs: man:openvpn(8)
           https://community.openvpn.net/openvpn/wiki/Openvpn23ManPage
           https://community.openvpn.net/openvpn/wiki/HOWTO
  Process: 5852 ExecStart=/usr/sbin/openvpn --daemon ovpn-%i --status /run/openvpn/%i.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/%i.conf --writepid /run/openvpn/%i.pid (code=exited, sta
 Main PID: 5856 (openvpn)
    Tasks: 1 (limit: 512)
   CGroup: /system.slice/system-openvpn.slice/openvpn@server.service
           └─5856 /usr/sbin/openvpn --daemon ovpn-server --status /run/openvpn/server.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/server.conf --writepid /run/openvpn/server.pid

May 03 15:30:05 openvpn2 ovpn-server[5856]: /sbin/ip addr add dev tun0 local 10.8.0.1 peer 10.8.0.2
May 03 15:30:05 openvpn2 ovpn-server[5856]: /sbin/ip route add 10.8.0.0/24 via 10.8.0.2
May 03 15:30:05 openvpn2 ovpn-server[5856]: GID set to nogroup
May 03 15:30:05 openvpn2 ovpn-server[5856]: UID set to nobody
May 03 15:30:05 openvpn2 ovpn-server[5856]: UDPv4 link local (bound): [undef]
May 03 15:30:05 openvpn2 ovpn-server[5856]: UDPv4 link remote: [undef]
May 03 15:30:05 openvpn2 ovpn-server[5856]: MULTI: multi_init called, r=256 v=256
May 03 15:30:05 openvpn2 ovpn-server[5856]: IFCONFIG POOL: base=10.8.0.4 size=62, ipv6=0
May 03 15:30:05 openvpn2 ovpn-server[5856]: IFCONFIG POOL LIST
May 03 15:30:05 openvpn2 ovpn-server[5856]: Initialization Sequence Completed
```

You can also check that the OpenVPN `tun0` interface is available by typing:

```bash
ip addr show tun0
```

You should see a configured interface:

```text
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 100
    link/none 
    inet 10.8.0.1 peer 10.8.0.2/32 scope global tun0
       valid_lft forever preferred_lft forever
```

If everything went well, enable the service so that it starts automatically at boot:

```bash
sudo systemctl enable openvpn@server
```

## Step 10: Create Client Configuration Infrastructure

Next, we need to set up a system that will allow us to create client configuration files easily.

### Creating the Client Config Directory Structure

Create a directory structure within your home directory to store the files:

```bash
mkdir -p ~/client-configs/files
```

Since our client configuration files will have the client keys embedded, we should lock down permissions on our inner directory:

```bash
chmod 700 ~/client-configs/files
```

### Creating a Base Configuration

Next, let's copy an example client configuration into our directory to use as our base configuration:

```bash
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf
```

Open this new file in your text editor:

```bash
vi ~/client-configs/base.conf
```

Inside, we need to make a few adjustments.

First, locate the `remote` directive. This points the client to our OpenVPN server address. This should be the public IP address of your OpenVPN server. If you changed the port that the OpenVPN server is listening on, change `1194` to the port you selected:

{% code-tabs %}
{% code-tabs-item title="~/client-configs/base.conf" %}
```text
. . .
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote server_IP_address 1194
. . .
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Be sure that the protocol matches the value you are using in the server configuration:

{% code-tabs %}
{% code-tabs-item title="~/client-configs/base.conf" %}
```text
proto udp
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Next, uncomment the `user` and `group` directives by removing the ";":

{% code-tabs %}
{% code-tabs-item title="~/client-configs/base.conf" %}
```text
# Downgrade privileges after initialization (non-Windows only)
user nobody
group nogroup
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Find the directives that set the `ca`, `cert`, and `key`. Comment out these directives since we will be adding the certs and keys within the file itself:

{% code-tabs %}
{% code-tabs-item title="~/client-configs/base.conf" %}
```text
# SSL/TLS parms.
# See the server config file for more
# description.  It's best to use
# a separate .crt/.key file pair
# for each client.  A single ca
# file can be used for all clients.
#ca ca.crt
#cert client.crt
#key client.key
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Mirror the `cipher` and `auth` settings that we set in the `/etc/openvpn/server.conf` file:

{% code-tabs %}
{% code-tabs-item title="~/client-configs/base.conf" %}
```text
cipher AES-128-CBC
auth SHA256
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Next, add the `key-direction` directive somewhere in the file. This must be set to "1" to work with the server:

{% code-tabs %}
{% code-tabs-item title="~/client-configs/base.conf" %}
```text
key-direction 1
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Finally, add a few commented out lines. We want to include these with every config, but should only enable them for Linux clients that ship with a `/etc/openvpn/update-resolv-conf` file. This script uses the resolvconf utility to update DNS information for Linux clients.

{% code-tabs %}
{% code-tabs-item title="~/client-configs/base.conf" %}
```text
# script-security 2
# up /etc/openvpn/update-resolv-conf
# down /etc/openvpn/update-resolv-conf
```
{% endcode-tabs-item %}
{% endcode-tabs %}

If your client is running Linux and has an `/etc/openvpn/update-resolv-conf` file, you should uncomment these lines from the generated OpenVPN client configuration file.

Save the file when you are finished.

### Creating a Configuration Generation Script

Next, we will create a simple script to compile our base configuration with the relevant certificate, key, and encryption files. This will place the generated configuration in the `~/client-configs/files` directory.

Create and open a file called `make_config.sh` within the `~/client-configs` directory:

```bash
vi ~/client-configs/make_config.sh
```

Inside, paste the following script:

{% code-tabs %}
{% code-tabs-item title="~/client-configs/make\_config.sh" %}
```text
#!/bin/bash

# First argument: Client identifier

KEY_DIR=~/openvpn-ca/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Save and close the file when you are finished.

Mark the file as executable by typing:

```bash
chmod 700 ~/client-configs/make_config.sh
```

## Step 11: Generate Client Configurations

Now, we can easily generate client configuration files.

If you followed along with the guide, you created a client certificate and key called `client1.crt` and `client1.key` respectively by running the `./build-key client1` command in step 6. We can generate a config for these credentials by moving into our `~/client-configs` directory and using the script we made:

```bash
cd ~/client-configs
./make_config.sh client1
```

If everything went well, we should have a `client1.ovpn` file in our `~/client-configs/files` directory:

```bash
ls ~/client-configs/files
```

{% code-tabs %}
{% code-tabs-item title="Output" %}
```text
client1.ovpn
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Transferring Configuration to Client Devices

We need to transfer the client configuration file to the relevant device. For instance, this could be your local computer or a mobile device.

While the exact applications used to accomplish this transfer will depend on your choice and device's operating system, you want the application to use SFTP \(SSH file transfer protocol\) or SCP \(Secure Copy\) on the backend. This will transport your client's VPN authentication files over an encrypted connection.

Here is an example SFTP command using our client1.ovpn example. This command can be run from your local computer \(OS X or Linux\). It places the .ovpn file in your home directory:

### Step 12: Install the Client Configuration {#step-12-install-the-client-configuration}

Now, we'll discuss how to install a client VPN profile on Windows, OS X, iOS, and Android. None of these client instructions are dependent on one another, so feel free to skip to whichever is applicable to you.

The OpenVPN connection will be called whatever you named the .ovpn file. In our example, this means that the connection will be called client1.ovpn for the first client file we generated.

### Windows

#### Installing

The OpenVPN client application for Windows can be found on[ OpenVPN's Downloads ](https://openvpn.net/index.php/open-source/downloads.html)page. Choose the appropriate installer version for your version of Windows.

### OS X

#### Installing

[Tunnelblick](https://tunnelblick.net/) is a free, open source OpenVPN client for Mac OS X. You can download the latest disk image from the [Tunnelblick Downloads](https://tunnelblick.net/downloads.html) page. Double-click the downloaded .dmg file and follow the prompts to install.

Towards the end of the installation process, Tunnelblick will ask if you have any configuration files. It can be easier to answer No and let Tunnelblick finish. Open a Finder window and double-click client1.ovpn. Tunnelblick will install the client profile. Administrative privileges are required.

### Step 13: Test Your VPN Connection {#step-13-test-your-vpn-connection}

Once everything is installed, a simple check confirms everything is working properly. Without having a VPN connection enabled, open a browser and go to [DNSLeakTest](https://www.dnsleaktest.com/).

The site will return the IP address assigned by your internet service provider and as you appear to the rest of the world. To check your DNS settings through the same website, click on Extended Test and it will tell you which DNS servers you are using.

Now connect the OpenVPN client to your Droplet's VPN and refresh the browser. The completely different IP address of your VPN server should now appear. That is now how you appear to the world. Again, DNSLeakTest's Extended Test will check your DNS settings and confirm you are now using the DNS resolvers pushed by your VPN.

### Step 14: Revoking Client Certificates {#step-14-revoking-client-certificates}

Occasionally, you may need to revoke a client certificate to prevent further access to the OpenVPN server.

To do so, enter your CA directory and `re-source` the vars file:

```bash
cd ~/openvpn-ca
source vars
```

Next, call the `revoke-full` command using the client name that you wish to revoke:

```bash
./revoke-full client3
```

This will show some output, ending in `error 23`. This is normal and the process should have successfully generated the necessary revocation information, which is stored in a file called `crl.pem` within the `keys` subdirectory.

Transfer this file to the `/etc/openvpn` configuration directory:

```bash
sudo cp ~/openvpn-ca/keys/crl.pem /etc/openvpn
```

Next, open the OpenVPN server configuration file:

```text
sudo nano /etc/openvpn/server.conf
```

At the bottom of the file, add the `crl-verify` option, so that the OpenVPN server checks the certificate revocation list that we've created each time a connection attempt is made:

{% code-tabs %}
{% code-tabs-item title="/etc/openvpn/server.conf" %}
```text
crl-verify crl.pem
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Save and close the file.

Finally, restart OpenVPN to implement the certificate revocation:

```bash
sudo systemctl restart openvpn@server
```

The client should now longer be able to successfully connect to the server using the old credential.

To revoke additional clients, follow this process:

1. Generate a new certificate revocation list by sourcing the vars file in the `~/openvpn-ca` directory and then calling the `revoke-full` script on the client name.
2. Copy the new certificate revocation list to the `/etc/openvpn` directory to overwrite the old list.
3. Restart the OpenVPN service.

This process can be used to revoke any certificates that you've previously issued for your server.



