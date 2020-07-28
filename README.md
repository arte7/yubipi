# yubiPI

The Aim is to make a pretty holistic security setup with our yubikeys

## Set up Yubikey with GPG (for SSH / git commit signing etc)

### First: what could Possibly go wrong??
- backup your secret GPG key, store it safely, may the force be with you

`$gpg --export-secret-key --armor AABBCCDD`

then:
create a set of subkeys, you may need them
for:
- signing [S]
- authentication [A]
- encryption [E]

(internet says 2048bit is the max keylength you can have on an old yubikey. 4+5 support over 2048)

`$gpg --edit-key AABBCCDD`

push your subkeys to your Yubikey
this is a destructive move,
once they're on a yubikey you can't ever copy them again.
Yubikeys are write only.

delete your private masterkey from your machine, only keep the public one

***Funfact:*** creating a new key + subkeys + rev cert. + associating them with the yubikey and their correct role
is a thing of seconds if you do it via the `--card-edit` menu.

Congrats! You have set up your yubikeys with GPG :)

#### Learnings

 - We used a guide to [create the perfect gpg key pair](https://alexcabal.com/creating-the-perfect-gpg-keypair), and overenthusiastically moved the master GPG keypair to a USB.
 When we needed to then move our subkeys to the yubikey, we encountered an error `secret key not available` OOPS - cue reimporting the secret key back to our laptops so we can successfully move the subkeys to the yubikey!
 **Important** only delete the GPG secret key after you are finished moving the subkeys to the yubikey!

### Now to use it to sign your git commits:

in Git you can add your public gpg key, to have it associated with your Account.

***Funfact*** if you already have your GPG key associated with your account and created the subkeys afterwards and then try to associate the key with it's new subkeys to your git account, you get a pretty misleading errormessage, wich asserts you already uploaded this particular key with all it's subkeys... so delete the key and associate it again.

 `git config --global user.signingkey YOURKEYIDEA`
(remove the --global if you want the key to be repository specific)

use git commit with `-S` to indicate that you want to sign the commit.
Then, you can enter in your yubikey pin.

***protip:*** Your yubikey should be inserted some seconds before commiting, otherwise it will give you nasty error messages

### Speaking of the yubikey pin...

It's not a good idea to stick with the default admin pin and user pin.

`gpg --card-edit` will allow you to edit the details for the yubikey, such as name and pin.

***Jet another "Funfact"*** We followed through what happens if you fail setting your PIN (bad idea)
and ended up screwing everything, cardreset did kind of work but somehow the keys were still related to the card and couldn't be stored on another time, so I created a new pair of keys <3

## Set up Yubikey for PAM authentication

PAM authentication stands for Pluggable Authentication Module (not to be confused with Privalleged Access Management). ELI5 - it is a authorization tool that can be configured (hence pluggable), and does basic auth and password management. [This guide](https://www.linux.com/news/understanding-pam/) gives a fairly good introduction to it if you want more information :)

**Important**: Enabling full disk encryption (FDE) with FileVault is highly recommended when using the macOS Login Tool. If you do not enable FDE, it is possible to reboot the Mac into recovery mode and disable the 2FA requirement.

**Important 2**: Not applicable for Catalina OS yet!

***protip:*** enable fingerprint recognition incase you break something you can still log in with you fingerssss
or just shut down and disable screensaver PAM again after you broke everything

### prerequisites
- download the [macOS Logon tool](https://developers.yubico.com/yubico-pam/Releases/pam_yubico-2.26.pkg)
- Download the [Yubikey Manager](https://www.yubico.com/products/services-software/download/yubikey-manager/)
- insert key, open manager and configure long touch (slot 2) with challenge response ***note:*** **reqiure touch** seems to not work with older keys when you later try to connect them to your account
- repeat with all the keys you want to have as backups
- then insert key and run: `ykpamcfg -2`
- repeat with all the keys to associate with your account

- first try out your PAM on screensaver `vi /etc/pam.d/screensaver`
- insert in after the last auth line `auth       required       /usr/local/lib/security/pam_yubico.so mode=challenge-response`
- save, lock mac, see if it works - good luck!

***Trouble!!!*** in case you had to set up your challenge multiple times and already associated run `rm /Users/username/.yubico/challenge-AABBCCDD` to remove the file and associate new

### It's getting serious!
- first: put on your cozy socks, they might prevent you from getting your feet cold
- `vi /etc/pam.d/authorization`
- Insert after the last auth line `auth       required       /usr/local/lib/security/pam_yubico.so mode=challenge-response`
- save have fun!

***in the end*** there is [a nice script to uninstall the logon tool](https://support.yubico.com/solution/articles/15000012625-uninstalling-the-macos-login-tool)

## Set up 2F authentication(Amazon, Facebook etc.)

- Download [Yubico authenticator](https://www.yubico.com/products/services-software/download/yubico-authenticator/)
- Enable 2FA with authenticator app instead of mobile device
- have your Yubikey inserted
- A QR code should show up open the Yubico App and scan the code, and add if you like the settings
- Repeat with all your backup keys 

***protipp*** in case you just ordered a backupkey two minutes ago, bc the other one is too old for the authenticator app just safe the QR code and add your othe key later.

- Finish setup 

***protipp*** in case your Yubikey 5C isn't compatible with your iPhone but you still wanna do Amazonshopping on mobile you can suppress the OTP on that specific browser the next time you log in from it.


## In case of emergency:
### manual reset...

- go to `gpg --card-edit` and cry for help
there are several options how to handle keyloss etc
