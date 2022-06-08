# Proxmox Custom Install Guide

This guide is meant to help those setting up a Proxmox server and are having specific problems along the way.

# Black screen and no GUI?

To get the installer to even start properly you need to do the following:

1. press the 'e' key when 'Install' option is highlighted.
1. In the GRUB menu replace the `quiet splash = silent` with `nomodeset`.
1. Now press F10 on your keyboard to continue booting.
1. Once the terminal says something about Xorg being init, you can hit ctrl+c on your keybaord to abort the install.
1. Now you will need to do the following:

```
Xorg -configure
mv /xorg.conf.new /etc/X11/xorg.conf
# edit the "Device" drive to be 'fbdev'
vim /etc/X11/xorg.conf
startx
```

Once you run startx, the GUI should appear. Install Proxmox and then reboot or shutdown.

_NOTE: You can persist this change by editing /etc/default/grub setting `GRUB_CMDLINE_LINUX_DEFAULT` to have `nomodeset` at the end._

# Proxmox Hardening

Proxmox has a couple of things that we may want to tighten down a bit more.

## Create a New User/Admin Account

- Use the following commands in the Proxmox shell:

```
adduser <username>
usermod -aG sudo <username>
```

- Now we have a new user. We create new ssh keys on another computer:

```
mkdir .ssh
mkdir .ssh/pve
ssh-keygen -t ed25519 -a <rounds> -f .ssh/pve
scp -P 22 .ssh/pve/id_ed25519.pub <username>@<ip_addr>:~
```

- Now ssh into the `<username>` you created. Append the id_ed25519.pub file contents to the .ssh/authorized_keys files using:
  - `cat ~/id_ed25519.pub >> .ssh/authorized_keys`

You can then delete the ~/id_ed25519.pub file and logout. Be sure to test that your SSH key before proceeding.

_NOTE: After creating the user in the shell, you can initialize the user in the proxmox web UI._

## Modify SSH

SSH is turned on by default, but we need to harden it a bit.

1. Stop the ssh service using:
   - `systemctl stop ssh.service` or
   - `service ssh stop`
1. Edit the `/etc/ssh/sshd_config` in the following ways:
   - Comment out 'Include...'
   - `Port <NEW_PORT>`
   - `DenyUsers root`
   - `AllowUsers <NON_ROOT_USERS>`
   - `PermitRootLogin no`
   - `PubkeyAuthentication yes`
   - `PasswordAuthentication no`
   - `PermitEmptyPasswords no`
   - `ChallengeResponseAuthentication yes`

To change the port on the server, you need to edit the `/etc/services` file and change the port your `<NEW_PORT>`.

Be sure to restart and re-enable the service again to apply the changes.

## UFW Firewall

_(Optional - May Cause VM Network Problems)_

While I have not entirely tested if the firewall was the problem or not, you have to be careful about enabling a CLI firewall.

1. Install ufw `apt install ufw`
1. Allow ssh and the web-UI ports:

- `ufw allow <NEW_PORT>`
- `ufw allow 8006`

You can restrict these to certain IPs if you wish.

# VMs

Setting up a VM is pretty straight forward. The only challenging parts are the hardware configurations.

## Windows

- Be sure to set the CPU Type to 'host' for the highest performance.
- Be sure that TMP 2.0 is enabled for Windows 11 support.

### GPU Passthrough

1. Add your GPU PCIe device after creating the VM.

## Linux (Ubuntu/Debian)

### GPU Passthrough

# Containers

The containers tutorial below this point will primarily be for hosting server applications and deep learning sandboxing.

## GPU Passthrough

## CUDA Containers for DL

nvidia-containers?

## OpenVPN Container
