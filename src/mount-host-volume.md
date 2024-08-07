# Mount host volume in LXC

This part of the guide is about mounting a directory from the host to a unprivileged container. The directory can be a SMB or a NFS share that is already mounted on the host, or any other local directory.

## Mount the directory

Edit `/etc/pve/lxc/<lxc-id>.conf` with your favorite editor. Mine is `nano`, BTW.

Add something like following example,

```config
mp0: /mnt/DigitalMemory/,mp=/mnt/DigiMem
mp1: /<path on the host>/,mp=/<path in the container>
```

Note: Do not put a space after the comma, it breaks the config file.

## Setup permission in the host

Now, let's set up the directory permission on the host machine.

First, some background knowledge. An unprivileged LXC container runs as a normal user with a unprivileged user id in the host machine (Let's refer to this normal user as the agent user, and the group that normal user is in as the agent group). A privileged container will have its root user has the same user id as the host root user, which makes it considered less secure. In our case, we are using a unprivileged container, and what we need to do is to **allow the agent group to access the desired directory in the host machine**. Luckily, we can predict the agent group's id based on its id in the container.

The formula is as follow,

$\text{GroupID}_{agent} = \text{GroupID}_{container} - 100000$

If we would like to have the group with id 12345 to access the directory on the host, we need to give access permission to group with id of 912345 in the host.

So, in the host machine, 

```bash
groupadd -g 154321 lxc_shares
```

In the container,

```bash
groupadd -g 54321 lxc_shares
```

Note the name of the group does not need to be the same in host and container, but to make one's life easier in the future, it is good to keep them the same.

Now, let's set up the permission of the directory in the **host**.

```bash
chown -R root:lxc_shares /mnt/DigitalMemory
```

To validate if it is successfully set in the host, one need to make sure that a user that is not the owner (in this case, `root`), but inside the `lxc_shares` group, can properly access the directory.

```bash
useradd foo
usermod -aG lxc_shares foo
# ...Test the access, like creating and deleting files.
userdel foo
```

If all good, we can proceed to the next step.

## Setup permission in the LXC container

Inside the LXC container,

```bash
useradd -m bar
groupadd -g 12345 lxc_shares
usermod -aG lxc_shares bar
```

After, `su bar`, one should be able to access the mounted directory at `/mnt/DigiMem`.

Note: If you do not like the default `/bin/sh` shell for the new user, as the root, you can do,

```bash
chsh bar
# Then typing /bin/bash or whatever shell you like
```

And, that is it, EZ, right?

Let's move on the next part.
