# archivesys - archival backup based on rsync

## Capabilities
- local or remote (using `rsync://` protocol against rsyncd) backup
- configureable backup exclusions (using rsync exclusions file)
- full or differential backups
- supports named backup sets for repetitive archiveal activities (and for differential backup)
- optional compression for data in flight
- log to stdout _and_ logfile (in `/var/log`)

## Requirements
- bash
- grep
- GNU coreutils (head, sha256sum, tail, tee)
- rsync v3.2.3 or higher
- sudo access (for log output and archive revision recording in `/var/log`)

## Examples
```bash
# Full archival backup of user's $HOME to remote server running rsyncd
archivesys -e ~/Documents/bckexclusions -f -n userbak -s $HOME rsync://someserver/backup

# Differential backup to local USB drive
archivesys -n userdiff -s $HOME /mnt/usbdrive
```
## Tips ##
Use the backup set naming feature (-n myname) to keep track of archives of identical volumes/file systems/paths. This will enable you to leverage rsync's differential capability.

If at all possible, _prefer to send your archival backups to a target that is either a locally mounted volume OR a remote sever running an rsync daemon_. If data security is not an issue, you can use the `rsync://` protocol; otherwise use rsync over SSH.

Avoid, if possible, sending archival backups with rsync to a network-oriented file system (e.g. NFS). In my testing, using a standard 25GB $HOME volume, _an rsync archive to an NFS mount took a number of hours, whereas using rsync:// protocol to a remote server took a number of minutes (depending on your uplink bandwidth)_. In using network-oriented file systems like NFS, you incur overhead of syncing coppies of the data that are written to the NFS mount, among all machines that have the NFS volume mounted. Effectively, you are "double mirroring" between the network file system's clients, and the source-target comparison behavior of rsync, which slows down rsync's copy rate considerably.

If possible, consider using differential backups (ommit the `-f` command line parameter) in rsync. This will permit rsync to only re-write data that has changed from _previous_ backup sets to the newly created archive target, making the archive process much quicker and more space efficient. The mechanics of how this works (using hard links) can be studied [here](https://jeelabs.org/article/1552a/).

## ** DISCLAIMER **
Use of this tool is provisional to the following user consent items: 
- **By using it, you agree to to assume all risk associated with it**. 
- **I am not responsible for failed backups, data corruption or any other issues arising from use of this software.** 
