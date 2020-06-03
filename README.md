# A(nsible) to Z(FS): Experimenting with Ansible against ZFS on Ubuntu
The playbook in this repository will walk you through managing ZFS on `localhost`. 

## Preparation
You may wish to run this on a disposable ZFS file system. ZFS supports using a file as a ZFS file system. The documentation suggests using `mkfile` but this is not available by default on Ubuntu these days. We have `dd` to the rescue!
The provided `hosts` file is to support using Ansible on localhost. 

### Set up a disposable ZFS file system
```
# dd if=/dev/zero of=/fast/zvolfile bs=1M count=1000
1000+0 records in
1000+0 records out
1048576000 bytes (1.0 GB, 1000 MiB) copied, 4.49232 s, 233 MB/s
# sudo zpool create volume0 /fast/zvolfile 
# zfs list
NAME      USED  AVAIL  REFER  MOUNTPOINT
volume0  85.5K   864M    24K  /volume0
```

[![asciicast](https://asciinema.org/a/336260.svg)](https://asciinema.org/a/336260)


## Ansible + ZFS
I assembled `zfs-management.yml` to test performing different ZFS management tasks. This makes creating volumes, setting quotas and mountpoints and managing other ZFS properties *super* easy. 
### Learn about the ZFS volume
First, we will use `zfs_facts` to gather some information about our volume.
Then, we will use that information to find out about disk usage across zpool and report it out.
Finally, we set some properties.

### Set mountpoint and refquota
In the task below, we set the mountpoint for `volume0/home/lucio` to `/home/lucio`:

```
        - name: Set mount point and home directory quota for Lucio
          zfs:
            name: volume0/home/lucio
            state: present
            extra_zfs_properties:
                    mountpoint: /home/lucio
                    refquota: 2G 
```

Changing that mountpoint is as simple as modifying the `mountpoint:` line in the playbook and re-executing the playbook.
Here is the output of `df` after the task as it is written above:

```
# df -h | grep lucio
volume0/home/lucio     864M     0  864M   0% /home/lucio
```
Let's move this user's home directory over to `/srv/data/lucio` by rewriting the task to:

```
        - name: Set mount point and home directory quota for Lucio
          zfs:
            name: volume0/home/lucio
            state: present
            extra_zfs_properties:
                    mountpoint: /srv/data/lucio
                    refquota: 2G
```

Run the playbook using `ANSIBLE_NOCOWS=1 v/bin/ansible-playbook  -i hosts zfs-management.yml`. 

```
# df -h | grep lucio
volume0/home/lucio     864M     0  864M   0% /srv/data/lucio
```

Done! That ZFS volume is now mounted at `/srv/data/lucio`.

## Put it all together
```
ANSIBLE_NOCOWS=1 v/bin/ansible-playbook  -i hosts zfs-management.yml
```

While cows are fun, your terminal can get crowded. I use `ANSIBLE_NOCOWS=1` to make the cows go away. 

## Resources
### Ansible Modules
[zfs](https://docs.ansible.com/ansible/latest/modules/zfs_module.html)

[zfs_facts](https://docs.ansible.com/ansible/latest/modules/zfs_facts_module.html)

[zpool_facts](https://docs.ansible.com/ansible/latest/modules/zpool_facts_module.html)

