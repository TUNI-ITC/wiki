
# Install TUNI VPN

The TUNI intra documentation is here https://intra.tuni.fi/en/handbook?page=2638 but its kind of a mess and therefore below easy to follow information.

## Self-maintained Linux Box (tested Ubuntu 18.04)

eduVPN application is not available for Linux and thefore you must use openvpn. Generic instructions are available here https://eduvpn.tuni.fi/vpn-user-portal/documentation for various OS, but, for example, Ubuntu is described below.

Install OpenVPN:

```
~$ sudo apt-get install network-manager-openvpn-gnome
```

Generate your own at https://eduvpn.tuni.fi/vpn-user-portal/configurations (give name, e.g., "joni-laptop", and store somewhere).

Start openvpn:

```
~$ sudo openvpn <PATH_TO_MY_GEN_CONFIG>.ovpn
```
And that's it - your internet connection goes over VPN and you have access to university computers!!

**Note:** You need to update the VPN configuration files every now and then (the expriry date by default is in the filename, e.g., "Downloads/eduvpn.tuni.fi_internet_20201102_joni-laptop.ovpn#)

## Update on the Linux desktop client of eduVPN: 

There is a Linux desktop client and Python API for eduVPN and here is the link to that: https://python-eduvpn-client.readthedocs.io/en/master/introduction.html#installation

I tested it on Ubuntu 18.04 and 20.04 and it works fine. The steps are simple and listed below (taken from the above documentation link). 
```
$ sudo -s
$ apt install apt-transport-https curl
$ curl -L https://repo.eduvpn.org/debian/eduvpn.key | apt-key add -
$ echo "deb https://repo.eduvpn.org/debian/ stretch main" > /etc/apt/sources.list.d/eduvpn.list
$ apt update
$ apt install eduvpn-client
```
Open the eduvpn-client from the terminal (or app launcher) and add the university name from the drop-down menu. They may ask you to sign-in with the TUNI username. Then select the university name and toggle the connected button. If it does not work then you are on your own. 

<!--
**Note** :
1. Check the name of the university before putting it in the app (They tend to change it frequently)
2. Before installing, check whether this eduVPN is still in use in our university or not (^)
-->
