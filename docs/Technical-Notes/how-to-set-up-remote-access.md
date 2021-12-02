# How to Set up Remote Access

_This is the guide for setting up the remote connection to your office machine and it will not tell you how to set up the machine itself (e.g. install Ubuntu, allocate disk space). If you would like some guidance on this topic, for example, you would want to make your remote machine to "look" like the rest of our machines, please refer to [this unofficial guide](https://github.com/v-iashin/TuniSurvivalKit/blob/master/how_to_setup_a_desktop.md) – also, let us know if it was useful and whether the wiki might benefit from having it here._

!!! warning
    Since this guide relies on `ssh-forward.tuni.fi` as a proxy server, you may only
    set up the remote connection if you are a **staff member** of the university, i.e.
    a student cannot apply for access to `ssh-forward.tuni.fi`.
    If you are a student, please contact the IT-Helpdesk and ask if they can
    give you access to `ssh-forward.tuni.fi`.

A bit of motivation and how it will work. There is no _official_ way of conneting remotely to a self-maintained machine. Here is how we can work around this problem. Your compute machine can be added to a research network `pit.cs.tut.fi` which practically means that it will have a fixed IP (or FQDN) and you will not need to type your credentials every 24 hours. However, another problem here is that this research network can only be reachable using university-maintained devices which uses a university WiFi or a pre-installed VPN. The solution to this problem is to use a proxy ssh-server (`ssh-forward.tuni.fi`) when connecting to your machine. This ssh-forwarding server is open to the public internet and from there you can reach `pit.cs.tut.fi`.

We assume that you have an Linux desktop (**host**) at your office and you would like to access it remotely from any device, e.g. your laptop (**client**). Here we provide both: how to setup the host and the client sides. If the host is already set up and you would like to just learn how to connect to it, [follow the guide for a client](#how-to-set-up-the-client-eg-your-laptop).


## How to Set up the **Host** (e.g. compute machine)?

1. Email `it-helpdesk@tuni.fi` and ask them to connect your office machine to `pit.cs.tut.fi` network. They will assign a fixed IP/FQDN and you will not need to type your credentials every 24 hours to have an internet connection. Specify the following information:
    - the inventory number of the machine (on the sticker),
    - MAC address of the socket in the machine you would like to use for the wired connection to the internet (you may have several Ethernet ports – you need only one),
    - mention the Ethernet socket number from the wall that you will use.
2. At this point, you should have had received the response from `it-helpdesk@tuni.fi` and be able to connect to the internet using the socket you specified. If so, check your IP and type `host your_IP` in your terminal to find out the FQDN of the machine. In our case, it was be something like `<IP.reversed> pointed to **********.pit.cs.tut.fi`.
3. Install `openssh-server` package on your host machine (via e.g. `sudo apt-get install openssh-server`). This will allow `ssh` connection to this machine.
4. Next, make sure **no** WiFi connection connects automatically after the startup. On Ubuntu, type `sudo nm-connection-editor` in your terminal (or just go to `Edit connection` from the status menu on Ubuntu 16.04).
5. Also, allow your connection (by default called `Wired connection`) to automatically connect when available.

Now your machine (**host**) should be reachable from TUNI-maintained computers connected to `TUNI-STAFF` WiFi or a pre-installed VPN directly via `ssh user@**********.pit.cs.tut.fi`. If you are using a non-university computer, you need to use the ssh-forwarding server (`ssh-forward.tuni.fi`) to reach `pit.cs.tut.fi`. Use the following guide to set up the connection to the forwarding server.

## How to Set up the **Client** (e.g. your laptop)?
Even though university-maintained devices can reach `pit.cs.tut.fi` without any proxy from university premises, we found that using the proxy even on university-maintained devices provides a more uniform experience. Otherwise you need to keep in mind which network you are using each time you ssh to your machine and adjust the CLI command accordingly (see the tip in the end of this section on how to configure an `ssh` command to use proxy by default when connecting to a specific device).

1. To connect to your compute machine from a self-maintained/personal device, you need to get access to the ssh-forwarding server `ssh-forward.tuni.fi`. For this, proceed to [id.tuni.fi/idm](https://id.tuni.fi/idm/?uiLang=en) -> `My user rights` -> `Apply for a new user right` -> if required, select your *Staff* contract -> search for `Linux Servers (LINUX-SERVERS) Personnel SSH tunneling service` and select it. In `Application details` put something like `For pit.cs.tut.fi connections`. Then, go to the `Applications` tab and wait until the access is granted (1 min).
2. `ssh-forward.tuni.fi` is open to the public internet and it requires 2-factor authentication. For this, log in to the forwarding server using your TUNI credentials (`ssh tuni_user@ssh-forward.tuni.fi`) **while being connected to one of the University networks (`roam.fi/eduroam/TUNI-STAFF` or a university VPN). If 2FA was not initialized, it will reject your password. Unfortunately, you cannot initialize 2-factor authentication from any network. You need to be physically at University and be connected to one of the networks there**.
3. Once you logged in to `ssh-forward.tuni.fi`, type `google-authenticator`. It will ask you several questions and show a QR code (resize your window to see it). Answer the questions as follows:
    - `Do you want authentication tokens to be time-based...` -> y
    - `Do you want me to update your "/home/user/...google_authenticator" file` -> y
    - `Do you want to disallow multiple uses...` -> n
    - `By default, tokens are good for 30 seconds...` -> n
    - `If the computer that you are logging into...` -> y
4. Once QR code is shown, install some 2-factor authenticator app on your smartphone if you don't have one yet e.g. from [Microsoft](https://www.microsoft.com/en-us/account/authenticator), [Authy](https://authy.com/), or [Google](https://www.google.com/search?q=Google+Authenticator+apple+android). The app will be used for two-step authentication when you connect from a non-university network is used e.g. your home internet. Once it is done, it will create an entry with a 6-digit passcode which changes every 30 secs.
5. Test your 2FA by trying to connect to `ssh-forward.tuni.fi` from a non-university network (e.g. using internet shared from your cell-phone) (`ssh your_tuni_username@ssh-forward.tuni.fi`).
6. Now you can connect to your machine from your local terminal jumping through the ssh proxy. Here is a convenient command: `ssh -J your_tuni_username@ssh-forward.tuni.fi your_host_username@*********.pit.cs.tut.fi` (`-J` means "jump" using the specified proxy)
    - If you are on a University network (`roam.fi/eduroam/TUNI-STAFF` or VPN), it will only ask for your TUNI and host passwords;
    - If you are using a non-University network, it will first ask you for a `Verification code` which is a temporal code from the 2FA app you installed on your smartphone. Then, you will need to type your TUNI and host passwords.

!!! tip
    Config the `ssh` connection in `~/.ssh/config`:
    ``` python
    Host connection_name
      HostName ***********.pit.cs.tut.fi
      User your_host_username
      ProxyCommand ssh your-tuni-username@ssh-forward.tuni.fi -W %h:%p
    ```
    After doing this, you will be able to do `ssh connection_name` to `ssh` directly to `*********.pit.cs.tut.fi`, forward ports, and transfer large files using `scp/rsync`. It is also useful if you are using `VSCode` or any other text editor which supports remote development. Additionally, it is handy if you would like to mount folders from the host to your client. You can use `sshfs connection_name:/path/to/remote_folder /path/to/local_folder`.

## Known Issues
- At the moment, only a staff member (doctorate students are staff members usually) can use
`ssh-forward.tuni.fi` proxy and, therefore, benefit from this set up. Please, let us know if you found
a way around it.
- You will need to contact `it-helpdesk@tuni.fi` and ask to activate and configure your wall internet socket in a special way. This might take some time.
- `ssh-forward.tuni.fi` doesn't support key-pair authentication.
- Depending on the network you are using, a different log in procedure will be required: only your TUNI password if you are connected to `roam.fi/eduroam/TUNI-STAFF`; and 2FA + your TUNI and host-machine passwords  on other networks.
- `ssh-forward.tuni.fi` has very limited disk space for each user (few MB). Therefore, can only be used as a proxy for your `ssh` connection *which is its main purpose*.
