+++
title = "How to Overcomplicate Offline Storage"
author = ["Walker Griggs"]
description = "What started as a 1 terabyte external harddrive loaded with a few sentimental photos has turned into a 40TB NAS and 26TB worth of offline drives. This post is an attempt to answer 'how I store and track offline files.'"
date = 2023-04-01
categories = ["devlogs"]
draft = false
creator = "Emacs 29.3 (Org mode 9.6.15 + ox-hugo)"
weight = 2016
featured_image = "img/how_to_overcomplicate_offline_storage/lto.jpg"
+++

Seven years ago, I made the decision to keep offline backups of all my personal data. What started as a 1 terabyte external harddrive loaded with a few sentimental photos, zipped folders of school projects, and maybe the odd 360p DVD rip has turned into a 40TB NAS and 26TB worth of offline drives.

{{< figure src="/img/how_to_overcomplicate_offline_storage/lto.webp" caption="<span class=\"figure-number\">Figure 1: </span>LTO Tapes: the dream no one can reasonably afford" width="420px" >}}

Recently, I tried explaining my system to a friend but my thoughts kept running off in every possible direction. This post attempts to answer 'how I store and track offline files'; [Data Preservation, Alf’s Room, and Spicy P](https://walkergriggs.com/2023/03/25/data_preservation_alfs_room_and_spicy_p/) answers 'why.'


## What to backup {#what-to-backup}

Before anyone thinks about how they handle offline storage, they should first think about what they’re storing. Personally, I have three rules.

_**Can this data be replaced?**_

If not, I won't hesitate to keep multiple offline copies. Data in this category includes family photos and artifacts of hard work like blog posts, source code, or research notes. These are items that need to be protected at all times.

_**Do I access the file regularly?**_

If I access the file less often than I perform a round of disk maintenance, it’s probably not worth keeping on a spinning disk. That said, pulling out the drives, setting up an external drive dock, and mounting the drives all take time. I avoid it when I can.

_**Am I running out of online storage space?**_

As I’ve expanded my networked storage, my definition of a “large file” has changed, and my tolerance for always-online storage has grown. Mileage may vary.


## Picking the right drive {#picking-the-right-drive}

Everyone has a different opinion on which drives are best for offline storage. I think the best drives are the ones you have.

Personally, my archive drives are a graveyard of systems-past. They’re all different capacities, manufactured by different companies, and spin at different speeds. As long as they’re above their [S.M.A.R.T. thresholds](http://ntfs.com/disk-monitor-smart-attributes.htm), they’re fine by me.

Capacity is another story; some drives are just too small to bother with. My general rule of thumb is “4 times the size of your data set, divided by your maximum acceptable number of drives”. Personally I never want to maintain more than 10 offline drives at a time. Routine scrubbing and maintenance can be a slow process; let’s not make it slower than it has to be.

For me, currently, that’s `(12TB * 4) / 10` or about 5TB per drive. At the moment, I have a grab bag of 8, 4, and 3TB drives, so that math works out pretty well.

Why 4x the size of the dataset? Well, hard drives – especially old drives – aren’t meant to be long term, offline data solutions. As a result, I try to keep 2 copies of every file spread across multiple drives. I also like to keep 50% capacity available, which is probably overkill but it spreads the files out nicely and reduces the blast radius should a drive completely fail.


### Storing your drives {#storing-your-drives}

This bit might sound pedantic, but storing your drives properly is, in many ways, just as important as their S.M.A.R.T. status. I keep my drives in [Orico cases](https://www.orico.cc/us/product/detail/3665.html), labeled with their model number, serial number, capacity, and canonical name.

The cases claim to be shock and static resistant, but I still keep the drive in an antistatic bag with a small silica packet inside the case just to be safe. Of course, I store those cases somewhere dry with little temperature fluctuation. Harddrives are pretty hearty, but I’d rather be safe than sorry.


## Workflow, in theory {#workflow-in-theory}

This is where things get opinionated and tailored to your needs. Personally I wanted a system that could:

-   Be fully administered (backup, recovery, and maintenance) in a Linux shell – bonus points if that shell is live booted off the thumb drive with only standard kernel tools
-   Leave the files fully intact and accessible without any special software or de-serialization or de-fragmentation
-   Locate the file among all the offline drives
-   Validate the content of every block with checksums
-   Replicate the files easily and implicitly

I chose `btrfs` and `git-annex`.

I initially narrowed my filesystem choice down to either BTRFS or ZFS, but I have personal experience with the former and dislike the experience of exporting and offlining ZFS pools. BTRFS is now included in the Linux kernel and has all the features I look for in a modern filesystem. Relevant to this use case: block-level deduplication, disk defragmenting, and data scrubbing.

Git Annex surprised me, honestly. I hadn’t given it much thought in the past, but it covered my requirements fully. At the very least, it aligns with my normal software development workflow. From their website:

> git-annex allows managing large files with git, without storing the file contents in git. It can sync, backup, and archive your data, offline and online. Checksums and encryption keep your data safe and secure. Bring the power and distributed nature of git to bear on your large files with git-annex.

Annex supports quite a few remote repository backends like web, bittorrents, XMPP, and S3 to name a few. Unless I decide to move files into AWS S3 or Glacier in the future, I’ll only ever use the bare filesystem. I’d recommend at least reading through their docs – they’re wonderful!


## Workflow, in practice {#workflow-in-practice}

My workflow, in practice, is pretty simple.

1.  Load a drive up with files until it’s mostly full
2.  Annex those files into a git repository and sync to the origin remote
3.  Defragment the drive
4.  Scrub the filesystem to ensure that all checksums match
5.  Clone the repository on another drive and copy over any files that have less than 2 copies.

Here are some rough steps to reproduce that workflow.


### Create an annex repository {#create-an-annex-repository}

First thing’s first, we need to create an annex repository. I always keep one local repository on my system, so I can run a quick git annex whereis when I need to locate a file. That local repo won't store any large files though.

```bash
$ mkdir -p ~/anenx && cd ~/annex
$ git init
$ git annex init
$ git annex numcopies 2
```


### Format and prepare a fresh drive {#format-and-prepare-a-fresh-drive}

Once we’ve set up the origin remote we can format our first drive, clone a copy of the annex, and create an “offline repo.”

A small but critical detail: name and label your drive the same thing. I mean psychically label the drive, initialize the filesystem, title offline annex, and mount to a directory all with the same name – drive0, in this case. You can use the annex description for a bit more metadata; I use the hdd model number, so I always have it tracked.

```bash
$ mkfs.btrfs -L drive0 /dev/sdb

$ mkdir -p /mnt/drive0
$ mount /dev/sdb /mnt/drive0 && cd /mnt/drive0

$ git clone ~/annex && cd ~/annex
$ git annex group drive0 archive
$ git annex wanted drive0 standard
$ git annex describe drive0 "WD80EMZZ-00TBGA"

$ git annex info drive0
uuid: 3ddb6b63-33d8-43fa-8553-e594866530af
description: WD80EMZZ-00TBGA0 [drive0]
trust: semitrusted
remote: drive0
cost: 100.0
type: git
repository location: /mnt/drive0/annex/
last synced: 2023-03-21 04:33:17 UTC
remote annex keys: 1
remote annex size: 355.86 megabytes
```


### Copy the files over {#copy-the-files-over}

At this point, we’re ready to start backing up our files! Rsync is the easiest way to copy our files over, but any tool that validates the files’ checksums will work.

```bash
$ cd /mnt/drive0/annex
$ rsync -h --progress /mnt/somefarawayland/big_buck_bunny.mp4 ./

$ git annex add ./big_buck_bummy.mp4
$ git commit -S -m "🅱️ig 🅱️uck 🅱️unny"
$ git annex sync
```

Syncing the file is a bit like pushing. It makes sure that all other remotes are aware of the commit and file. If we jump back to the origin repo and sync as well, we should see the newly committed file.

```bash
$ cd ~/annex
$ git annex sync

$ git annex list
here
|drive0
||web
|||bittorrent
||||
_X__ big_buck_bunny.mp4
```

What's more, since we set the desired number of copies to 2, we can list all files missing a copy. We can even check the location of the file and grab some info on the drive – all the info you’d ever need when recovering offline data.

```bash
$ git annex list --lackingcopies 1
here
|drive0
||web
|||bittorrent
||||
_X__ big_buck_bunny.mp4

$ git annex whereis big_buck_bunny.mp4
whereis big_buck_bunny.mp4 (1 copy)
    0eb61bd8-80d8-4ebe-8ba8-2f93296ac1ad -- wgriggs@DebianBox:/mnt/drive0/annex [drive0]
ok
```

You might notice as well that if you list the repo, the files have been moved into a `.git/annex` directory and replaced by a symlink. This structure lets you organize your files -- no matter how large -- into directories which span multiple harddrives.

```bash
$ ls -l
big_buck_bunny.mp4 -> .git/annex/objects/some/long/path.mp4
```

If, for example, I uploaded `tears_of_steel.mp4` in this same repository but on another drive, each drive would have a symlink to both files. But the links would only resolve to the actual files on each respective drive.

```bash
$ git annex list --lackingcopies 1
here
|drive0
||drive1
|||web
||||bittorrent
|||||
_X___ big_buck_bunny.mp4
__X__ tears_of_steel.mp4

$ ls -al /mnt/drive0
big_buck_bunny.mp4 -> .git/annex/objects/some/long/path.mp4
tears_of_steel.mp4 -> .git/annex/objects/some/other/path.mp4
```

This use of symlinks is one of the hidden gems in git-annex. Every offline drive stores a local copy of the repository, and the content of the repository contains every symlink. Only the drive that contains the file has a valid link, but the drive knows the location of every file.

Was your laptop with the origin repository remote stolen? Not a problem: grab any offline drive and clone the repo. The caveat here is that you need to keep the repositories on each drive up to date with git annex sync, but that shouldn’t be an issue if you perform routine maintenance.


### Duplicate the files {#duplicate-the-files}

Now that one of our drives has the file annexed, we need to create a second copy. Setup a second drive same as before: create the filesystem, mount the drive, and clone the repository. This time, instead of using rsync to copy the files to the new drive, we can use annex get to identify files on available drives missing duplicates.

```bash
$ git annex get --auto --numcopies 2
```

After the file has been copied over the second drive and the repos have syncd, you'll see that a copy exists on both drives.

```bash
$ git annex list
here
|drive0
||drive1
|||web
||||bittorrent
|||||
_XX__ big_buck_bunny.mp4
```


### Maintenance {#maintenance}

Let’s say this drive has the capacity for exactly one copy of `big_buck_bunny.mp4`. Before packing the drive away, I’ll run a quick disk defragmentation and data scrub to validate the file’s integrity. This step is likely overkill, but what about this process isn’t?

```bash
$ btrfs filesystem defrag -vf /mnt/drive0
$ btrfs scrub /mnt/drive0
```

An important note: I’ll pull this drive out every 4-6 months and run another scrub! Part of offline storage is actually spinning the disk up, mounting it, and making sure all the files are as you expect.
