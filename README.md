# Work-in-Progress Copy DO NOT USE #

## btrfs-borg
Btrfs-borg makes snapshots and backs up a list of btrfs snapshotted subvolumes using Borg.


The theory of btrfs-borg is to abstract subvolume names from their underlying filesystems.  You won't need to worry about the underlying structure.  By default, it backs up all subvolumes that aren't snapshots, including the special rootfs subvolume used by LX containers.  These containers are automatically renamed the basename of the container (eg: lxc-info -n $name).  Btrfs-borg will search for named subvolumes on all volumes, and if duplicates are found will prefix the basename device (eg: sda2, sdb3, etc) to the snapshot name, so that @subvol1b will not clobber @subvol1a when restoring from backup.

*TODO: delete a lot of the stuff that follows*

It will also optionally back up a list of directories without first creating snapshots for them.  It includes special support for containers managed by LXC's btrfs backend.  Finally it supports building manifests of directories that won't be backed up.  If you configure manifest support, please make sure that the target location of the manifest will be on one of the subvolumes that you back up.

eg: if SUBS='@home rootfs', but you also have a subvolume called @backups mounted at /home/backups, then 1) /home/backups will not be part of your borg backup, because btrfs-subvolume snap does not back up child subvolumes.  2) If your manifest goes to /home/backups/manifest then it will not be backed up.  If /home/backups is not a separate subvolume, then /home/backups/manifest will be part of the btrfs snapshot and part of the borg backup.

I tag releases before doing something experimental and try to keep disruptive changes on the devel branch.

At some point I plan split it into a script+config file, because most people prefer to configure once and enable features as they need them, rather than reconfiguring with every new release...

## Notice
Btrfs-borg is designed to operate on named subvolumes underneath a volume
that has been mounted without the subvol= or subvolid= options.
Usually / is mounted with something subvol=@, subvol=rootfs, or something
similar.

To prepare your system for btrfs-borg, create a mountpoint like this:
    `install -d -m 700 /btrfs-admin`

If you do not want to see a new directory in your /, do this instead:
    `install -d -m 700 /.btrfs-admin`

Then, for example, make the following addition to /etc/fstab:
```
#
# / was on /dev/sda1 during installation
#
# UUID=a91471b4-26be-43f3-9fdf-7b9342a0c4ea /    btrfs defaults,subvol=@ 0 1
####  /\/\ duplicate this line, and remove the subvol argument, eg: \/\/  ###
# UUID=a91471b4-26be-43f3-9fdf-7b9342a0c4ea /btrfs-admin  btrfs defaults 0 1
#
# or for a hidden /.btrfs-admin:
# UUID=a91471b4-26be-43f3-9fdf-7b9342a0c4ea /.btrfs-admin btrfs defaults 0 1
```

It creates snapshots at /btrfs-admin/snapshots/subvolume_name, or
/btrfs-admin/volume_name/snapshots/subvolume_name, as appropriate.
For LX containers, /btrfs-admin/snapshots/lxc-container_name, or
as appropriate /btrfs-admin/volume_namesnapshots/lxc-container_name.
In any case, it removes the snapshots used to make the backup after
the backup completes successfully.

Seconds since epoch will allow you to make backups as often as
possible, with no archive name collisions.  I chose not to use a
format like 2016-01-12-17:03:09 because `borg list <repo>` already
outputs this format in the right-hand column, and someone might find
machine-readable date stamps useful for his/her custom pruning script.
P.S. seconds since epoch is cool.  Additionally, some filesystems
don't handle the colon ":" very well.  Finally, it's shorter than the
format (date, time, timezone) yet provides the same information.  
eg: 2017-02-05T14:41:19-0500  
vs  1486323679

Get the current date in seconds since epoch: `date +%s`  
Convert seconds since epoch  
    to localised format: `date -d @1486323679`  
    to RFC 2822: `date -R -d @1486323679`  
    to ISO 8601: `date -I -d @1486323679`  
Convert to a custom format: `date +%F-%R -d @1503865189`

man date(1) for more info.
