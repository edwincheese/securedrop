Installing SecureDrop
=====================

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [Terminology](#terminology)
  - [App Server](#app-server)
  - [Monitor Server](#monitor-server)
  - [Source Interface](#source-interface)
  - [Document Interface](#document-interface)
  - [Journalist Workstation](#journalist-workstation)
  - [Admin Workstation](#admin-workstation)
  - [Secure Viewing Station](#secure-viewing-station)
  - [Two-Factor Authenticator](#two-factor-authenticator)
  - [Transfer Device](#transfer-device)
- [Before you begin](#before-you-begin)
  - [Computers](#computers)
  - [USBs/DVDs/CDs](#usbsdvdscds)
  - [Passphrases](#passphrases)
- [Set up the Secure Viewing Station](#set-up-the-secure-viewing-station)
  - [Create a GPG key for the SecureDrop application](#create-a-gpg-key-for-the-securedrop-application)
  - [Import GPG keys for journalists with access to SecureDrop](#import-gpg-keys-for-journalists-with-access-to-securedrop)
- [Set up the Admin USB](#set-up-the-admin-usb)
- [Set up the Servers](#set-up-the-servers)
- [Install SecureDrop](#install-securedrop)
  - [Install Ansible](#install-ansible)
  - [Clone and verify the release code](#clone-and-verify-the-release-code)
  - [Set up SSH keys for the Admin](#set-up-ssh-keys-for-the-admin)
  - [Gather the required information](#gather-the-required-information)
- [Testing the Installation](#testing-the-installation)
  - [Test connectivity](#test-connectivity)
  - [Test the web application](#test-the-web-application)
- [Journalist's Workstation Setup](#journalists-workstation-setup)
  - [Set up journalist GPG keys](#set-up-journalist-gpg-keys)
  - [Journalist Logging In](#journalist-logging-in)
    - [Using Tails](#using-tails)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

This guide outlines the steps required to install SecureDrop 0.3.

When running commands or editing configuration files that include filenames, version numbers, admin or journalist names, make sure it all matches your setup.

## Terminology

A number of terms used in this guide, and in the [SecureDrop workflow diagram](https://freedom.press/securedrop-files/SecureDrop_complex.png), are specific to SecureDrop. The list below attempts to enumerate and define these terms.

### App Server

The *Application Server* (or *App Server* for short) runs the SecureDrop application. This server hosts both the website that sources access (*Source Interface*) and the website that journalists access (*Document Interface*). You may only connect to this server using Tor.

### Monitor Server

The *Monitor Server* keeps track of the *App Server* and sends out an email alert if something seems wrong. You may only connect to this server using Tor.

### Source Interface

The *Source Interface* is the website that sources will access when submitting documents and communicating with journalists. This site is hosted on the *App Server* and can only be accessed over Tor.

### Document Interface

The *Document Interface* is the website that journalists will access when downloading new documents and communicating with sources. This site is hosted on the *App Server* and can only be accessed over Tor.

### Journalist Workstation

The *Journalist Workstation* is a machine that is online and used together with the Tails operating system on the *online* USB stick. This machine will be used to connect to the *Document Interface*, download documents, and move them to the *Secure Viewing Station* using the *Transfer Device*

### Admin Workstation

The *Admin Workstation* is a machine that the system administrator can use to connect to the *App Server* and the *Monitor Server* using Tor and SSH. The administrator will also need to have an Android or iOS device with the Google Authenticator app installed.

### Secure Viewing Station

The *Secure Viewing Station* (or *SVS* for short) is a machine that is kept offline and only ever used together with the Tails operating system on the *offline* USB stick. This machine will be used to generate GPG keys for all journalists with access to SecureDrop, as well as to decrypt and view submitted documents.

Since this machine will never touch the Internet or run an operating system other than Tails on a USB, it does not need a hard drive or network device. We recommend physically removing the drive and any networking cards (wireless, Bluetooth, etc.) from this machine.

### Two-Factor Authenticator

There are several places in the SecureDrop architecture where two-factor authentication is used to protect access to sensitive information or systems. These instances use the standard TOTP and/or HOTP algorithms, and so a variety of devices can be used to provide two factor authentication for devices. We recommend using one of:

* An Android or iOS device with [Google Authenticator](https://support.google.com/accounts/answer/1066447?hl=en) installed
* A [Yubikey](http://www.yubico.com/products/yubikey-hardware/)

### Transfer Device

The *Transfer Device* is the physical media used to transfer encrypted documents from the *Journalist Workstation* to the *Secure Viewing Station*. Examples: a dedicated USB stick, CD-R, DVD-R, or SD card.

If you use a USB stick for the transfer device, we recommend using a small one (4GB or less). You will want to securely wipe the entire device at times, and this process takes longer for larger devices.

Depending on your threat model, you may wish to only use one-time use media (such as CD-R or DVD-R) for transferring files to and from the SVS. While doing so is cumbersome, it reduces the risk of malware (that could be run simply by opening a malicious submission) exfiltrating sensitive data, such as your private encryption key or the contents of decrypted submissions.

## Before you begin

You will need the following inventory of hardware items for the installation. For specific hardware recommendations, see the [Hardware Guide](hardware.md).

### Computers

* Application Server
* Monitor Server
* Admin Workstation (any spare computer that can be connected to the firewall and can run Tails)

### USBs/DVDs/CDs

* Ubuntu Live USB/CD/DVD (to install Ubuntu on the servers)
    * To create the Ubuntu Live USB/DVD/CD, see [this section in the Ubuntu Install Guide](/docs/ubuntu_config.md#creating-the-ubuntu-installation-media).
* Tails Live USB/CD/DVD (to set up Tails Live USB's with persistence)
    * To create the Tails Live USB, see the [Tails Guide](/docs/tails_config.md).
* One *Transfer Device* for transferring files, marked *transfer*. For the installation, we recommend using a brand new USB drive.

Additionally, you will need a minimum of 3 USB sticks which will become Tails Live USB's with persistence. You should mark one *offline*, one *online*, and one *admin*. This is enough to set up a system with one admin and one journalist (note that the same person can perform both of these roles). To add more administrators or journalists, you will need more USB sticks.

Finally, each user, whether admin or journalist, will need a *Two-Factor Authenticator*.

Each journalist will also need a *Transfer Device* for transferring files between the *Secure Viewing Station* and their *Journalist Workstation*, and a personal GPG key. Make sure you [create GPG keys](/docs/install.md#set-up-journalist-gpg-keys) for journalists who do not already have one.

It is also recommended that you use an external hard drive to back up encrypted submissions and some form of removable media to back up the GPG keyring on the *App* server.

### Passphrases

**TODO** Numerous components of the system require accounts...

* A secure and unique passphrase for the persistent volume on each of the two USB sticks

## Set up the Secure Viewing Station

The *Secure Viewing Station (SVS)* is a machine that is kept offline and only ever used together with the Tails operating system on the *offline* USB stick. Since this machine will never touch the Internet or run an operating system other than Tails on a USB, it does not need a hard drive or network device. We recommend that you physically remove the hard drive and networking cards, such as wireless and bluetooth, from this machine. If you are unable to remove a card, tape over it or otherwise physically disable it. If you have questions about using an old machine for this purpose, please contact us at securedrop@freedom.press.

### Create a GPG key for the SecureDrop application

When a document is submitted through the *Source Interface* on the *App Server*, the document is automatically encrypted with the SecureDrop Application GPG key. If you have not previously created a GPG key for SecureDrop, follow the steps below to do so now:

* Boot the Secure Viewing Station from the *offline* USB stick
* Open a terminal and run `gpg --gen-key`
* When it says, `Please select what kind of key you want`, choose `(1) RSA and RSA (default)`
* When it asks, `What keysize do you want?` type `4096`
* When it asks, `Key is valid for?` press Enter to keep the default
* When it asks, `Is this correct?` verify that you've entered everything correctly so far, and type `y`
* For `Real name` type: `SecureDrop`
* For `Email address`, leave the field blank and press Enter
* For `Comment` type `SecureDrop Application GPG Key`
* Verify that everything is correct so far, and type `o` for `(O)kay`
* It will pop up a box asking you to type a passphrase, but it's safe to click okay without typing one (since your persistent volume is encrypted, this GPG key is stored encrypted on disk)
* Wait for your GPG key to finish generating

To manage GPG keys using the Tails graphical interface, click the clipboard icon in the top right and choose "Manage Keys". You should see the key that you just generated.

![My Keys](images/install/keyring.png)

Select the key you just generated and click "File" and "Export". Save the key to the *Transfer Device* as `SecureDrop.asc`, and make sure you change the file type from "PGP keys" to "Armored PGP keys," which can be switched right above the 'export' button. This is the public key only.

![My Keys](images/install/exportkey.png)
![My Keys](images/install/exportkey2.png)

You'll need to verify the fingerprint for this new key during the `App Server` installation. Double-click on the new key you just generated and change to the `Details` tab. Write down the 40 hexadecimal digits under `Fingerprint`. (Your GPG key fingerprint will be different than what's in this photo.)

![Fingerprint](images/install/fingerprint.png)

### Import GPG keys for journalists with access to SecureDrop

While working on a story, journalists may need to transfer some of the documents or notes from the *Secure Viewing Station* to the journalist's work computer on the corporate network. To do this, the journalists need to decrypt the documents using the SecureDrop application's GPG key and re-encrypt them with their own keys. If a journalist does not already have a key, follow the steps above to create one. 

If the journalist does have a key, transfer the public key to the *Secure Viewing Station* using the *Transfer Device*. Open the file manager and double-click on the public key to import it. If the public key is not importing, rename the file to end in ".asc" and try again.

![Importing Journalist GPG Keys](images/install/importkey.png)

## Set up the Admin USB

In this section we will set up the Admin Live USB, a Tails Live USB with persistence.

Start by booting the *Secure Viewing Station (SVS)* from the Tails Live USB. To do this, insert the Tails Live USB and boot the machine. Open the boot menu and select the Tails Live USB from the menu.

(On the Intel NUCs, to reboot first press the power button to trigger shutdown. Once the machine has shut down, press the power button again to boot it. As it boots, hold F10 to get the boot menu.)

At the *Welcome to Tails* dialog, choose *No* for *More options?* and click *Login*. Once the desktop environment has loaded, insert the spare USB that will become a Tails live USB with persistence for the Admin Workstation.

Click the *Applications* menu in the top left corner of the screen, and open *Applications > Tails > Tails Installer*. Choose *Clone & Install* and select the USB stick you inserted as the Target Device. In my testing, it was automatically selected because it was the only other USB device inserted at the time. Click *Install Tails* and click *Yes* to continue. When the installer is done, click *OK* to close the program.

You will be able to set up persistence on this new Tails USB because it was created with the Tails Installer. Shut down the running Tails instance by clicking the power button icon in the top right corner of the screen and choosing *Shut down immediately*. Remove the original Tails USB, leaving the new Admin Live USB, and boot the Admin Workstation.

"Welcome to Tails", "More Options?" choose *No* and click *Forward*

Open the Persistence Wizard - click OK, configure the volume, select all options for storage, restart.

**TODO** these instructions are basically "how to create a Tails Live USB persistence." Should we:

1. Keep these here (I don't think so, they're sufficiently generic that they should go elsewhere)
2. Move them to the Tails doc (tails_config.md)
3. Delete them and refer people to the official Tails documentation

## Set up the Servers

Start by plugging the *Application Server* and the *Monitor Server* into the firewall.

Install Ubuntu 14.04.1 (Trusty) on both servers. For detailed information on installing and configuring Ubuntu for use with SecureDrop, see the [Ubuntu Install Guide](docs/ubuntu_config.md).

## Install SecureDrop

Connect the Admin Workstation's Ethernet port to the shared firewall. Boot the Admin Workstation with the Admin Tails USB that we created earlier. Make sure to type the password in for your persistence drive, but before clicking enter, also click 'yes' for more options. Click 'forward.'

You will be prompted to enter a root password. This is a one-time session password, so you will only be creating it for this one session. 

Open a terminal (click the terminal icon in the top menu).

### Install Ansible

First, you need to update your package manager's package lists to be sure you get the latest version of Ansible. It should take a couple minutes.

    sudo apt-get update

Now, install Ansible by entering this command:

    sudo apt-get install ansible

### Clone and verify the release code

Next, you will need to clone the SecureDrop repository:

    git clone https://github.com/freedomofpress/securedrop.git

Before proceeding, verify the signed git tag for this release.

First, download the *Freedom of the Press Foundation Master Signing Key* and verify the fingerprint. (**TODO** update with the airgapped signing key's fingerprint)

    gpg --keyserver pool.sks-keyservers.net --recv-key 9092EDF6244A1603DCFDC4629A2BE67FBD67D096
    gpg --fingerprint 9092EDF6244A1603DCFDC4629A2BE67FBD67D096

(**TODO** can we trust `--recv-key` or do we still have to manually verify the fingerprint, even when we use it to fetch the key in the first place?)

Now, verify that the current release tag was signed with the master signing key.

    cd securedrop
    git checkout 0.3
    git tag -v 0.3

You should see 'Good signature from "Freedom of the Press Foundation Master Signing Key"' in the output of `git tag`. If you do not, signature verification has failed and you *should not* proceed with the installation. If this happens, please contact us at securedrop@freedom.press.

### Set up SSH keys for the Admin

Now that you've verified the code that's needed for installation, you need to create an SSH key on the admin workstation. Initially, Ubuntu has SSH configured to authenticate users with their password. This new key will be copied to the *Application Server* and the *Monitor Server*, and will replace the use of the password for authentication. Since the Admin Live USB was set up with [SSH Client persistence](https://tails.boum.org/doc/first_steps/persistence/configure/index.en.html#index3h2), this key will be saved on the Admin Live USB and can be used in the future to authenticate to the servers in order to perform administrative tasks.

First, generate the new SSH keypair:

    $ ssh-keygen -t rsa -b 4096

You'll be asked to "enter file in which to save the key." Here you can just keep the default, so type enter. Choose a strong passphrase to protect the SSH private key. 

Once the key has finished generating, you need to copy the public key to both servers. Use `ssh-copy-id` to copy the public key to each server in turn. Use the user name and password that you set up during Ubuntu installation.

**TODO** This will ask us to verify the SSH fingerprints of each server. Do we want to verify these? The way to do it would be to add a step when installing Ubuntu on each server where we log in and do ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key, then record that value to compare here.

    $ ssh-copy-id <username>@<App IP address>
    $ ssh-copy-id <username>@<Mon IP address>

Verify that you are able to log in to both servers without being prompted for a password:

    $ ssh <username>@<app_ip_address>
    $ ssh <username>@<mon_ip_address>

Once you've verified that you are able to log in to each server, you can log out of each by typing this:

    $ logout # or Ctrl-D

### Gather the required information

Make sure you have the following information and files before continuing:

* The IP address of the *App Server*
* The IP address of the *Monitor Server*
* The SecureDrop application's GPG public key (from the *Transfer Device*)
* The SecureDrop application's GPG key fingerprint
* The email address that will receive alerts from OSSEC
* The GPG public key and fingerprint for the email address that will receive the alerts
* An image to replace the SecureDrop logo on the *Source Interface* and *Document Interface*
    * This will replace the SecureDrop logo on the source interface and the document interface.
    * Recommended size: `500px x 450px`
    * Recommended format: PNG
* The first name a journalist who will be using SecureDrop (you can add more later)
* The username of the system administrator

Next, you will have to change into the ansible-base directory in the SecureDrop repo that you cloned earlier:

    $ cd install_files/ansible-base

Copy the following required files to `securedrop/install_files/ansible-base`:

* Application GPG public key file
* Admin GPG public key file (for encrypting OSSEC alerts)
* Custom header image file.

It will depend what the file location of your USB stick is, but, for an example, if you are already in the ansible-base directory, you can just run: 

    $ cp /media/[USB folder]/SecureDrop.asc .

Then repeat the same step for the Admin GPG key and custom header image.

Next, you're going to edit the inventory file and replace the default IP addresses with the ones you chose for app and mon. Here, `editor` refers to your preferred text editor (nano, vim, emacs, etc.).

    $ editor inventory

After changing the IP addresses, save the changes to the inventory file and quit the editor.

Next, fill out `prod-specific.yml` with the Application Server IP and hostname, along with the same information for the Monitor Server (you should have had this information saved from when you installed Ubuntu).

**TODO**: more detailed instructions for filling out prod-specific.yml

    $ editor prod-specific.yml

Run the playbook. You will be prompted to enter the sudo password for each server. `<username>` is the user you created during the Ubuntu installation, and should be the same user you copied the ssh public keys to.

    $ ansible-playbook -i inventory -u <username> -K --sudo site.yml

The ansible playbook will run, installing SecureDrop and configuring and hardening the servers. This will take some time, and will return the Terminal to you when it is complete. If an error occurs while running the playbook, please submit a detailed [Github issue](https://github.com/freedomofpress/securedrop/issues/new) or send an email to securedrop@freedom.press.

Once the installation is complete, the hidden service addresses for each service will be placed in the following files in `install_files/ansible-base`:

* app-source-ths
* app-document-aths
* app-ssh-aths
* mon-ssh-aths

Update the inventory, replacing the ip addresses with the onion addresses:

    $ editor inventory

Use the values fetched from the app and monitor servers with Micah's Tails_files program.

## Testing the Installation

### Test connectivity

* ssh over tor into both servers
* sudo as root
* both web interfaces are available over tor
* OSSEC alert about OSSEC starting after reboot is received.
* run 'unattended-upgrades' reboot and run again
* run `uname -r` verify you are booted into grsec kernel
* `aa-status` shows the apparmor status
* `iptables-save` shows the current applied iptable rules

### Test the web application

1. Configure your torrc with the values with app-document-aths, app-ssh-aths, and mon-ssh-aths, and reload your Tor.
2. Make sure the source interface is available, and that you can make a submission.
3. SSH to the app server, `sudo su`, cd to /var/www/securedrop, and run `./manage.py add_admin` to create a test admin user.
4. Test that you can access the document interface, and that you can log in as the admin user you just created.
5. Test replying to the test submission.
6. Test that the source received the reply.
7. **TODO** More testing...

Once you've tested the installation and verified that everything is working, see [How to Use SecureDrop](/docs/user_manual.md).

## Journalist's Workstation Setup

The journalist workstation computer is the laptop that the journalist uses on a daily basis. It can be running Windows, Mac OS X, or GNU/Linux. This computer must have GPG installed.

### Set up journalist GPG keys

Each journalist must have a personal GPG key that they use for encrypting files transferred from the `Secure Viewing Station` to their `Journalist Workstation`. The private key, used for decryption, stays on their `Journalist Workstation`. The public key, used for encryption, gets copied to the `Secure Viewing Station`.

If a journalist does not yet have a GPG key, they can follow these instructions to set one up with GnuPG (GPG).

* [GNU/Linux](https://www.gnupg.org/gph/en/manual.html#AEN26)
* [Windows](http://gpg4win.org/)
* [Mac OS X](https://support.gpgtools.org/kb/how-to/first-steps-where-do-i-start-where-do-i-begin)

### Journalist Logging In

In order to view the `Document Interface`, journalists needs to either 1) install the Tor Browser Bundle and modify it to authenticate to the hidden service, or 2) modify Tor through their Tails operating system to accomplish the same task. The latter is highly recommended since many news organzation's corporate computer systems have been compromised in the past.

#### Using Tails

Each journalist that will be using Tails to connect to the `Document Interface` will need to install a persistent script onto their Tails USB stick. Follow [these instructions](/tails_files/README.md) to continue. In order to follow those instructions you'll need the HidServAuth values, which can be found in `install_files/ansible-base/app-document-aths`.

Once you have installed this script, the journalist will have to run it each time they boot Tails and connect to the Tor network in order to login to the `Journalist Interface`. A convenient launcher is automatically set up by `install.sh`, which can be double-clicked to automatically set up the Tor configuration.
