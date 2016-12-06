
# Minimalistic HowTo Create an Encrypted Filesystem in a File Container

Create the container file (adjust size as needed):

    dd if=/dev/zero bs=1M count=<count> of=./container.img
    cryptsetup luksFormat ./container.img
	cryptsetup luksOpen ./container.img container

Create a file system and mount:

    mkfs.ext4 /dev/mapper/container
	mkdir ./container
	mount /dev/mapper/container ./container
    #use filesystem
	umount ./container

Alternatively e.g. create a zfs volume in your container:

    zpool create tank /dev/mapper/container
	zfs create container/hideaway
	cd /container/hideaway
    #use filesystem
	zpool export container

Close encrypted container when done:

    cryptsetup luksClose /dev/mapper/container

Change passphrase (Dump key slots and check if there is room for another key. If yes add new key and remove old one):

    cryptsetup luksDump ./container.img
	cryptsetup luksAddKey ./container.img
	cryptsetup luksRemoveKey ./container.img


