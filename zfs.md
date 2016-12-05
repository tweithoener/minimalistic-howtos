# Minimalistic ZFS HowTo

## Create zpool

Check blockdevice

    lsblk -o NAME,PHY-SEC,LOG-SEC,UUID

Calculate optimal ashift VAL as log_2(secsize)

    zpool create -o ashift=<VAL> bagger /dev/disk/by-uuid/...

Set attributes as needed

    zfs set xattr=sa bagger
    zfs set compression=lz4 bagger

## Create a zfs filesystem

    zfs create bagger/files

Make snapshots visible

    zfs set snapdir=visible bagger/files

Before reboot (at least on ubuntu 16.04) disable MOUNT and SHARE in /etc/defaults/zfs.

## Useful Commands

Import existing zpools

    zpool import bagger
    zfs mount -a

Check status

    zpool list
    zpool status

Snapshots

    zfs snapshot -r bagger/files@`date +%Y%m%d-%H%M%S`
    zfs list -t snapshot
    zfs rollback bagger/files@snapshot-name

Syncing ZFS

    zfs send bagger/files@snapshot | zfs recv laster/files_clone
    zfs send -i bagger/files@snap1 bagger/files@snap2 | zfs recv laster/files_clone
    zfs send ... | ssh host2 zfs recv ...

Getting rid of stuff

    zfs destroy bagger/files@snapshot-name
    zfs destroy bagger/files
	zpool destroy bagger

Unmount

    zpool export bagger

