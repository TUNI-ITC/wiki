# How to Set up Remote Access

_This is the guide for setting up the remote connection to your office machine and it will not tell you how to set up the machine itself (e.g. install Ubuntu, allocate disk space). If you would like some guidance on this topic, for example, you would want to make your remote machine to "look" like the rest of our machines, please refer to [this unofficial guide](https://github.com/v-iashin/TuniSurvivalKit/blob/master/how_to_setup_a_desktop.md) – also, let us know if it was useful and whether the wiki might benefit from having it here._

We assume that you have an Ubuntu 16.04 or 18.04 desktop (**host**) at your office and you would like to access it remotely from, let's say, your personal laptop (**client**). If the machine is already set up and you would like to just learn how to connect to it, use only the guide for a client.

The connection will be maintained using a proxy server (`ssh-forward.cc.tut.fi`). Currently, this is the most convenient and stable way of setting up the remote connection to your compute machine (to the best of our knowledge – let us know if there is something else).

The host machine will be reachable using a University-maintained laptop using University `TUNI-STAFF` WiFi or a pre-installed TUNI VPN, for a self-maintained laptop you will need to do something extra and we have it in this guide.

## Known Issues

- You will need to contact `it-helpdesk@tuni.fi`. This might take some time.
- `ssh-forward.cc.tut.fi` doesn't support key-pair authentication. You can only login with passwords.
- Different log-in procedure depending on the network you are using. One-step on `roam.fi/eduroam/TUNI-STAFF`, two-step on other networks.
- `ssh-forward.cc.tut.fi` has very limited disk space for each user (few MB). Therefore, can only be used as a proxy for your `ssh` connection *which is its main purpose*.

## How to Set up the **Host** (e.g. compute machine)?
1. Email `it-helpdesk@tuni.fi` to connect your office machine to `pit.cs.tut.fi` network. They will assign a fixed IP/FQDN and you will not need to type your credentials every 24 hours to have an internet connection. Specify the following things:
    - the inventory number of the machine (on the sticker),
    - MAC address of the socket in the machine you would like to use for the wired connection to the internet (you may have several Ethernet ports--you need only one),
    - mention the Ethernet socket number from the wall that you will use.
2. At this point, you should have had received the response from `it-helpdesk@tuni.fi` and be able to connect to the internet using the socket you specified. If so, check your IP and type `host your_IP` to find out the FQDN. It should be something like `IP_reversed pointed to **********.pit.cs.tut.fi`.
3. Install `openssh-server` on your machine (host). This will allow `ssh` connection to this machine.
4. Next, make sure **no** WiFi connection connects automatically after the startup.  Type `sudo nm-connection-editor` in terminal (or just go to `Edit connection` from the status menu on 16.04).
5. Allow your `Wired connection` to automatically connect when available.

## How to Set up the **Client** (e.g. your laptop)?
If you have a machine which is already connected to `pit.cs.tut.fi`, you will only need to follow this guide.

1. Now your machine (host) should be reachable from TUNI-maintained computers connected to `TUNI-STAFF` WiFi or VPN directly via `ssh`. To connect to it from a self-maintained/personal device, you need to get access to `ssh-forward.cc.tut.fi`. For this, proceed to [tut.fi/omatunnus](https://www.tut.fi/omatunnus) (yes, `tut` not `tuni`) -> `Services` -> `System Access` (wait 10 sec) -> search for and select `ssh-forward.cc.tut.fi`. You will get the confirmation email shortly.
2. Initialize the two-step verification at `ssh-forward.cc.tut.fi`. For this, `ssh` with your TUNI credentials to `ssh-forward.cc.tut.fi` while being connected to one of the University networks (`roam.fi/eduroam/TUNI-STAFF` or a university VPN)--you cannot do it from any other network but [we got you here as well](#how-to-initialize-the-two-step-verification-remotely). Type `google-authenticator`. It will ask you several questions and show a QR code (resize your window to see it). Answer the questions as follows:
    - `Do you want authentication tokens...` -> y
    - `Do you want me to update your "/home/user/...` -> y
    - `Do you want to disallow multiple uses...` -> n
    - `By default, tokens are good for 30 seconds` -> n
    - `If the computer that you are logging into` -> y
8. Install `Google Authenticator` (or any other analogue from e.g. Microsoft) app for your smartphone. It is going to be used for two-step authentication when non-university network is used (e.g. connecting from home internet). You don't need an account to use this service if it will ask--just find a button to scan a QR code. Once it is done it will create an entry with a 6-digit passcode which changes every 30 secs.
9.  To connect to your machine, use `ssh -J your_tuni_username@ssh-forward.cc.tut.fi your_host_username@*********.pit.cs.tut.fi`
    - If you are on a University network (`roam.fi/eduroam/TUNI-STAFF` or VPN), it will only ask for your TUNI and host passwords;
    - If you are using a non-University network, it will also ask for a `Verification Code` which is a temporal code from `Google Authenticator` app you installed on your smartphone.


!!! tip
    Config the `ssh` connection in `~/.ssh/config`:
    ``` python
    Host connection_name
      HostName ***********.pit.cs.tut.fi
      User your_username_at_the_host_machine
      ProxyCommand ssh your-tuni-username@ssh-forward.cc.tut.fi -W %h:%p
    ```
    After doing this, you will be able to do `ssh connection_name` to `ssh` directly to `*********.pit.cs.tut.fi`, forward ports, and transfer large files using `scp/rsync`. It is also useful if you are using `VSCode` or any other text editor which supports remote development. Additionally, it is handy if you would like to mount folders from the host to your client. You can use `sshfs connection_name:/path/to/remote_folder /path/to/local_folder`.


## How to Initialize the Two-step Verification Remotely
If you are setting up your connection remotely and you don't have an access to the university networks nor VPN at the moment, you still can do it. For this, you will need to apply for another TUT service. Proceed to [tut.fi/omatunnus](https://www.tut.fi/omatunnus) (yes, `tut` not `tuni`) -> `Services` -> `System Access` (wait 10 sec) -> search for `linux-ssh.cc.tut.fi` or `staff-linux.cc.tut.fi` depending on whether your are a student or a staff. When the access is granted (5 mins), `ssh` to one of them, and from there `ssh` to `ssh-forward.cc.tut.fi` -- no verification code will be asked as these servers are in the university network.
