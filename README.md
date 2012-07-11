verifyBoot manual v.2012-02-03
==============================

home:                      https://github.com/najamelan/verifyBoot
please report any bugs at: https://github.com/najamelan/verifyBoot/issues

Introduction
------------

verifyBoot helps you to automate the process of checking if your MBR and a partition have not changed.
verifyBoot makes sha256 hashes, first of the entire partition, then of your MBR and every file individually on the partition to help to find out what has changed.

If you already know the background on this script and want to get started immediately, check the «verifyBoot» script which is a sample script for the evocation of «verifyBoot_internal», which is the real workhorse. Documentation is in «verifyBoot» and at the bare minimum the only thing you need to add is the uuid of the partition you want to verify.


Background
----------

Nowadays it has become reasonably trivial to have linux installed on an encrypted partition using dmcrypt or other encryption software. The usual procedure for system start-up implies for the bios to load the MBR of the boot device or the boot sector of a partition which then loads grub. Grub will load the kernel which will then take care of further booting and decryption of the encrypted system partition.

The problem with this approach is that everything in the «/boot» partition is not encrypted, facilitating a rootkit attack. An attacker could make the user run a process with the required privileges before or during a moment when the boot-medium gets plugged into the running operating system. This would allow them to place a rootkit, but also to access to the unencrypted data which would probably be what's at stake here in the first place, so this is not a very interesting scenario when considering the threat of a rootkit.

The second option an attacker has when they can't convince the user to run a rogue process is is to get physical access to the boot-medium. Since «/boot» is not encrypted, they can easily install a rootkit compromising everything on the system as soon as the user provides their pass-phrase.

It is possible to considerably raise the bar of such an attack by installing one's boot partition on a usb-stick that one carries at all time. This makes it hard for an attacker to obtain the unencrypted boot partition to compromise it. What do you do however when you go swimming, or when you get arrested or if you forget your usb stick somewhere. You could no longer be a 100% sure it wasn't compromised.

After you have been separated from your boot medium you should verify that it hasn't changed. This is where verifyBoot comes in. It helps you to automate that process of verification. Note that you will need a safe system to run the verification test from. This could be a verified live system that you obtain from a trusted source for example. If the boot system has changed you will need to create a new one from a trusted system or use an encrypted backup copy to restore your boot partition before booting from it.

Obtaining a safe system can be non-trivial for people with limited geek-connectivity. If it might be a little help this are the pgp fingerprints of some signing keys:

Debian CD signing key          -- DF9B 9C49 EAA9 2984 3258 9D76 DA87 E80D 6294 BE9B
Tails developers - signing key -- 0D24 B36A A9A2 A651 7878 7645 1202 821C BE2C D9C1

Note that this is not a secure source of the above fingerprints, but it may be an extra source permitting a little bit more means of verification as long as you obtain this document through a different channel as other sources for the keys or their fingerprints.

Note that it could still be possible for an attacker to use bios or embedded control chip code or hardware firmware to hack into a system. See the following for some examples:

https://en.wikipedia.org/wiki/Rootkit
http://arstechnica.com/security/news/2009/03/researchers-demonstrate-bios-level-rootkit-attack.ars
https://www.zdnet.com/blog/security/researchers-find-insecure-bios-rootkit-pre-loaded-in-laptops/3828


Also all risks of physical key-loggers, video-surveillance, network breach, etc remain.

In summary, to have your system set-up at this level of security the following steps are required:

1. install an encrypted Unix based system
2. have your /boot partition on a portable drive that you will always carry on you
3. edit fstab not to mount the /boot partition by default (add «,noauto» to the options). Not having this partition in your fstab on Debian means that one needs root privileges to mount it.
4. recommended: make your /boot folder immutable when your partition is not mounted. This will make updates on grub and kernel fail reminding you to mount your boot partition. You can use the program «sudo chattr +i /boot» (change attributes) for making the boot folder immutable. You can test with either «lsattr -d /boot» or «sudo touch /boot» which should now give a permission denied error.
5. Run verifyBoot on your boot medium after every time you have mounted it (except when you mount as read-only). Store the result somewhere safe or sign it with your pgp key and store it anywhere you want (possibly online).
6. When you get separated from your boot medium, before booting from it again, find a safe system to run verifyBoot on your boot medium.
7. Compare the hash with the stored hash and if it has changed, do not boot from your boot medium again. You can look at the detailed report produced by verifyBoot to see which files on your boot partition have changed. There aren't many innocent file on /boot so you shouldn't take the risk and recreate a new boot medium from a safe source or restore an encrypted backup.



Dependencies
------------
A standard Unix system should already have all that is needed:

bash
dd
mount
grep
blkid
sed
openssl dgst

Usage
-----
The actual script is verifyBoot_internal which takes up to 3 arguments. See the sample verifyBoot script for how to call it.


Known bugs and limitations
--------------------------

- Currently verifyBoot hashes the entire partition + MBR for a concise result (eg. one hash says it all) and then mounts the volume to hash every file individually and the first ext superblock in case of an ext file system. This means reading all data twice and there are certainly more efficient ways of doing this, but those aren't implemented and this being a bash script they might never be.
- Currently providing a mount point with a directory name that ends in a backslash '\' does not work. This might be a bug in mount and needs to be sorted out.
- If your partition is already mounted on several places, amongst which some in rw, which it shouldn't, error checking could be a bit sketchy. Has not been tested