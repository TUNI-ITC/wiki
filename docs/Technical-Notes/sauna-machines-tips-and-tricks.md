# Best Practices, Tips and Tricks for SAUNA Machines

This wiki page is a collection of tips and tricks, but also best practices for using the SAUNA machines. If you have a tip or trick that you would like to share, please add it to this page. If you are looking for a way to connect to SAUNA machines, please see [the guide for setting up the remote access](./how-to-set-up-remote-access.md).

## Storage

General idea of different disks

### SSD: boot disk

- *New machines have a partition of NVMe as boot disk*
- Contains
    1. root (/) dir
    2. /home dir
    3. user’s dir (/home/{user_name})
- Good practices:
    1. Store your (small) repos, personal data etc. here, under you own user dir.
    2. Do not store data or large files
    3. Space is limited and shared between users! Be mindful of that.

### NVMe: fast disk 💨

- Mounted to /home/nvme
    1. Newer machines contain a folder with user’s name **where users should place all their data**. Keeps everything neat.
- Good practices:
    1. Use this disk to run your experiments (and large repos)
    2. Use this disk to store your active data
    3. Space is limited and shared between users! Be mindful of that.

### HDD: storage disk

- Mounted to /home/hdd
    1. Some machines have multiple HDDs mounted to e.g. `/home/hdd1` and `/home/hdd2`
    2. Newer machines contain a folder with user’s name where users should place all their data. Keeps everything neat.
- Good practices:
    1. Use this disk to store (inactive) data.
    2. Space is limited and shared between users! Be mindful of that.
- Create folders with your own name and accumulate files under these! (If not done already)

    ```bash
    cd /home/nvme
    sudo mkdir ilpo
    sudo chown ilpo ilpo/
    ```

  - Now you can create files under this folder

## Upgrading and Updating the Machines

### Updating

❗ **Every user is responsible to keep their machines up to date** ❗

Every now and then, the machines need to be updated and upgraded. This is done by running the following commands:

```bash
sudo apt update
sudo apt upgrade
sudo apt autoremove
```

### Upgrading the Linux version

❗ **Do not do this if you are not certain of your actions** ❗

When you should upgrade the Linux version?

- When you are certain that you need a newer version of the Linux kernel (e.g. for a specific feature or a package)
- When support for the current LTS version is ending. LTS versions are supported for 5 years, but after that, you should upgrade to a newer version since older ones are not getting security updates. ➡️ Support for Ubuntu 18.04 LTS ends in April 30th 2023. After that, you should upgrade to Ubuntu 20.04 LTS or newer.

If you are certain that you want to upgrade the Linux version, you can follow these steps:

0. **Backup your data!** This is important since the upgrade process can fail and you might lose your data.

1. Check that you have the latest updates installed

    ```bash
    sudo apt update
    sudo apt upgrade
    sudo apt autoremove
    ```

2. Do a reboot

    ```bash
    sudo reboot
    ```

3. Check that you have screen installed. If your SSH connection is lost during the upgrade, screen will allow you to continue the upgrade from where you left off. Upgrade process will start a screen session automatically.

    ```bash
    sudo apt install screen
    ```

4. Install Ubuntu update tool

    ```bash
    sudo apt install update-manager-core
    ```

5. Make sure you can SSH to port 1022. Addittional sshd will be started on port 1022. If something goes wrong, you can still connect to the machine and continue the upgrade.

    ```bash
    sudo ufw allow 1022/tcp
    ```

6. Start the upgrade

    ```bash
    sudo do-release-upgrade
    ```

7. Reboot the machine

    ```bash
    sudo reboot
    ```

#### Additional resources

- Here is a nice guide for upgrading Ubuntu 18.04 LTS: [How to Upgrade Ubuntu 18.04 to 20.04](https://www.cyberciti.biz/faq/upgrade-ubuntu-18-04-to-20-04-lts-using-command-line/).
- How to reattach to the screen session if your SSH connection is lost: [See this serverfault post](https://serverfault.com/questions/387547/how-do-i-reattach-to-ubuntu-servers-do-release-upgrade-process).

## Development Environments

### Miniconda (Python)

Please see this video if you are unfamiliar with Conda:

[The only CONDA tutorial you'll need to watch to get started](https://youtu.be/sDCtY9Z1bqE?si=7BKNk9gAfNw8jPSk) (YouTube)

- Miniconda is installed **per user!**
- Install via terminal:

    [Installing on Linux — conda 24.1.3.dev33 documentation](https://conda.io/projects/conda/en/latest/user-guide/install/linux.html)

- Good cheat sheet for commands [here](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf)

!!! tip
    By default conda environments will be installed to the home directory of the user (`~/miniconda3/envs`). This is not ideal since the home directory is located on the SSD and the space is limited. Instead, you should install the environments to the NVMe disk, and make a symbolic link to the custom location to easily activate such environment. Here is how you can do it:

    ```bash
    # Create environment
    conda create --prefix /home/nvme/$USER/.envs/$MY_ENV python=3.9
    # Create symbolic link
    ln -s /home/nvme/$USER/.envs/$MY_ENV ~/miniconda3/envs/$MY_ENV
    # Activate environment
    conda activate $MY_ENV
    ```

### VSCode

Install VSCode to the client machine (your laptop) and use the Remote - SSH extension to connect to the SAUNA machines. This way you can use the full power of the SAUNA machines while having a nice development environment on your laptop. See [the guide how to connect to a remote host from VSCode](https://code.visualstudio.com/docs/remote/ssh#_connect-to-a-remote-host). It is recommended that you first add SAUNA machine to your SSH config file. See [the tip under How to setup client-section](./how-to-set-up-remote-access.md#how-to-set-up-the-client-eg-your-laptop).

## Adding New Sudo Users

1. First add the user

    ```bash
    sudo adduser new_user
    ```

2. Add the user to the sudo group

    ```bash
    sudo adduser new_user sudo
    ```

3. Create folders for the user

    ```bash
    cd /home/nvme
    sudo mkdir new_user
    sudo chown new_user new_user/

    cd /home/hdd
    sudo mkdir new_user
    sudo chown new_user new_user/
    ```

4. Share the username and password with the new user. Tell them to change the password after the first login.
