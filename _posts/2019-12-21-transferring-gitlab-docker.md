---
layout: post
title:  "Transferring Docker GitLab Repositories to a New Server"
date:   2020-01-15 20:00:00 +0800
tags: [docker,git]
navigation: true
---
In a recent move of offices, we had to stop our GitLab docker container and
decided to take the opportunity to improve the storage of our git repositories.
Instead of storing our git repositories on a desktop computer that's acting as a
server, we wanted to move them to a Network Access Storage (NAS).
This NAS is by [Synology](https://www.synology.com/) and has been set up with RAID to automatically backup data stored on the device,
providing us with better resilience against storage failure.
Additionally, it provides some optimisations in data transfer and storage, which
the standard desktop doesn't have.

However, the difficulty comes when trying to transfer the repositories as GitLab stores secrets to encrypt the repositories.
When searching for how to transfer the repositories, many different methods
appeared for the different possible GitLab setups.
Therefore, this post shows the process that we followed to successfully
transfer our repositiories from one GitLab docker container to another.

> This post assumes that the GitLab version is 12.6.

## Backup
The first step is to backup the repositories on the old server, which in this
case are stored on the host machine and bind-mounted into the GitLab docker container.
To do this, we must enter the container and use a gitlab command to generate the
backup as a tar file, storing it into a bind-mounted folder so that we can access it from the host machine.

First, enter the container:

```bash
docker exec -it <container id> bash
```

Second, create a folder for holding the backup and additional files that we need:

```bash
mkdir /var/opt/gitlab/gitlab-backup
```

Now, generate a backup:

```bash
gitlab-backup create
```

After that, locate the backup, which is found under the
`/var/opt/gitlab/backups` folder, and transfer it to the backup folder:

```bash
cp <backup name> /var/opt/gitlab/gitlab-backup
```

Now we need to find the additional files that I mentioned earlier. These are
**gitlab-config.rb** and **gitlab-secrets.json** files, which are found under the bind-mounted folder **/var/opt/gitlab**. Copy these files to the backup folder:

```bash
cp gitlab-config.rb /var/opt/gitlab/gitlab-backup

cp gitlab-secrets.json /var/opt/gitlab/gitlab-backup
```

## Transfer
Now that we have a backup of the GitLab repositories and the necessary configuration files, we can transfer these to the new machine.
There are several different ways the backup folder can be transferred to the new machine, we chose to transfer over the network as both our machines were connected to the same network.

To transfer the folder, you can use the **scp** command on the host machine as follows:

```bash
scp -r <location of /var/opt/gitlab on host machine>/gitlab-backup <new machine username>@<new machine ip address>:<~>
```

## New GitLab Container
On the new machine, we need to create a new empty GitLab docker container. The
docker image used must be the same version as the previous image on the old machine.

> Here I'm assuming the new machine has Docker installed. If not, please go ahead with installing Docker.

Create the necessary folders to persist the data on the host machine:

```bash
mkdir ~/gitlab/config
mkdir ~/gitlab/data
mkdir ~/gitlab/log
```

Next, download and run the image:

```bash
docker pull gitlab-ce:<tag>

docker run -d \
	-v ./gitlab/config:/etc/gitlab \
	-v ./gitlab/data:/var/opt/gitlab \
	-v ./gitlab/log:/var/log/gitlab \
	-p 80:80 \
	gitlab-ce:<tag>
```

> Wait here for the GitLab container to set itself up.

## Restore
With the new GitLab docker image setup and running, we can transfer our backup files into the container.

```bash
cp ~/gitlab-backup/<backup name> ~/gitlab/data/backups
cp ~/gitlab-backup/gitlab-config.rb ~/gitlab/config
cp ~/gitlab-backup/gitlab-secrets.json ~/gitlab/config
```

Finally, we can run the restore process inside the container:

```bash
docker exec -it <container id> bash

gitlab-backup restore
```

With that, the transfer of the GitLab repositories is complete.
For us, the process of backing up our GitLab repositories can be improved
further by establishing an offsite backup as well.
However, for now, moving our GitLab to a NAS with RAID setup has already significantly improved our resilience to storage failure.
