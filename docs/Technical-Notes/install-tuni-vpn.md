
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
