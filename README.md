# Ardexa VPN

This is a streamlined installer for the PiVPN project targeting ArdexaLinux and ArdexaTunnel.

PiVPN actually uses a generic suite of tools, so will work on any Debian-based
operating system and architecture.

## Setup

You MUST give your ArdexaLinux machine a Static IP address before attempting to
setup Ardexa VPN.  Please refer to the Ardexa Central documentation for details
on how to do this: https://ardexa.zendesk.com/hc/en-us/articles/360000328335

### Edit `setupVars.conf`.

The structure of this file is very strict.

---
:warning: **DO NOT use Notepad or Wordpad**.

For Windows users, please use a _modern_ text editor.  We recommend
* Atom: https://atom.io
* Notepad++
* Sublime Text
* VS Code
---

* **DO NOT** Use spaces anywhere
* **DO NOT** Change the character encoding (ASCII or UTF8 only)
* **DO NOT** Change the line endings (Windows users will need to be careful)

Replace `__IFACE__` with the Ardexa Linux interface name you wish to use with
the VPN. To get a list of interface names, run `ip a` via the REMOTE SHELL.

Replace `__STATIC_IP__` with the Static IP assigned to your chosen `__IFACE__`.
`ip a` will provide you with this IP address, or use the NETWORK tab on the
DEVICES page of the Ardexa Web App.

### Prepare the machine for install

Using REMOTE SHELL, run
```
mkdir /etc/pivpn/
```

Using SEND FILES, transfer the following files
  * dir: `/etc/pivpn` file: `setupVars.conf`
  * dir `/tmp` file: `setup-ardexa-vpn`

Using REMOTE SHELL, run
```
chmod +x /tmp/setup-ardexa-vpn
```

Make sure `ufw` is enabled
```
ufw status
```

If this reports `inactive`, run the following command
```
ufw enable
```
And then check the status again.

### Installation

Launch the install script using REMOTE SHELL. The whole process should take no
more than a few minutes, depending on your connection speed
```
/tmp/setup-ardexa-vpn
```

## Site Specific Routing
Once the installation is complete, there are some final tweaks required before
you can start issuing keys.

Using REMOTE SHELL, run:
```
echo route-nopull >> /etc/openvpn/easy-rsa/pki/Default.txt
```

Update the following command, replacing the variables to match your site
configuration, and then run it using REMOTE SHELL
```
echo route $NETWORK_ADDR $SUBNET_MASK >> /etc/openvpn/easy-rsa/pki/Default.txt
```

Where `$NETWORK_ADDR` is the first address of the network range. For
example:
```
echo route 172.16.10.0 255.255.255.0 >> /etc/openvpn/easy-rsa/pki/Default.txt
```

## Create OpenVPN Profile
The VPN is now ready for use. You can build an `ovpn` profile for use with any
OpenVPN client.

To create a new profile, run the following command. Replace `name` with the
name of the profile
```
pivpn add nopass -n name -d 1080
```
**NB** Please ignore any errors this returns.

Use GET FILES to download the new profile
```
/home/ardexa/ovpns/name.ovpn
```

## Preparing to use the VPN
We recommend the official OpenVPN client.

To install the OpenVPN Connect Client, follow the instructions here:

https://openvpn.net/client-connect-vpn-for-windows/

You will also need to have the `ardexa-tunnel` installed: https://ardexa.zendesk.com/hc/en-us/articles/360001052536-Installing-the-Tunnel-Client

## Open the VPN
* Browse to the TUNNEL tab for the device you are connecting to and enter `1194` for both the Remote and Local Ports. Leave the IP as `127.0.0.1`
* Click `Copy to Clipboard`
* Paste the command into a command shell, run the command and then enter your Ardexa Web App password
* Once the Ardexa Tunnel is connected, then double-click the `ovpn` file you downloaded earlier to launch the OpenVPN Client
* Check the box `Connect after import` and then click `Add`

Your VPN is now connected and you can access any IP on the remote network as if you were connected directly!
