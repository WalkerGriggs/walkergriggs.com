+++
title = "How to Overcomplicate Offline Storage"
author = ["Walker Griggs"]
date = 2023-03-15
categories = ["devlogs"]
draft = true
creator = "Emacs 27.2 (Org mode 9.4.4 + ox-hugo)"
weight = 2015
featured_image = "img/the_guy_who_likes_lemons/walker_griggs_the_guy_who_likes_lemons.jpg"
+++

Seven years ago, I made the decision to keep an offline backup of data I found personally important. What started as a 1 terrabyte external harddrive filled with a few sentimental photos, zipped folders of school projects, and maybe the odd 360p DVD rip has turned into a 40TB NAS (network attached storage) and 26TB offline drives.

I tried recently explaining my system to friend but my thoughts kept running off in every possible direction. This post answers "how I store and track offline files"; I'm drafting a post for 'why', but that's a substantially longer post.


## What to backup {#what-to-backup}

Before you think about _how_ you store offline files you first have to enumerate _what_ you're storing. Personally, I have three rules.

_**"Can this data be replaced?"**_

If not, I wont hesitate to keep _multiple_ offline copies. Data in this category includes family photos and artifacts of hard work like blog posts, source code, or research notes. These are items that need to be protected at all times.  Whether of not I keep a copy online is a matter of frequency.

Speaking of which, _**do I access the file than twice a year?**_

If not, it's probably not worth keeping on a spinning disk. Pulling out my drives, setting up an external drive dock, and mounting the drive all take time so I avoid it when I can.

_**Am I running out of online storage space?**_

This one depends on how much storage space you're working with. As I've expanded my networked storage, my definition of a "large file" has changed.


## Picking the right drive {#picking-the-right-drive}

Everyone has a different opinion on this one. Personally, my archive drives are a graveyard of systems-past. Each drive has a slightly different capacity, spin at a different speed, and are a different formfactor. As long as it's above its S.M.A.R.T. thresholds, it's fine by me.

What about the capacity, how large should the drives be? My general rule of thumb is "4 times the capacity of your data set, divided by your maximum accepible number of drives". Personally I never want to have more than 10 drives at a time. Routine scrubbing and maintenance can be a slow process; let's not make it slower than it has to be.

For me, currently, that's `(12TB * 4) / 10` or about 5TB per drive. At the moment, I have a grab bag of 8, 4, and 3TB drives, so that math works out pretty well.

Why 4x the size of the dataset? Well, hard drives -- especially old drives -- aren't meant to be long term, offline data solutions. As a result, I try to keep 2 copies of every file spread across all the drives. I also like to keep 50% capacity available, which is probably overkill but it spreads the files out nicely and reduces the blast radius should a drive completely fail.

My recommendation for sourcing the drives is "cheap and slow". If you can salvage the drives from you old systems, even better. I find that I can rotate my NAS drives 1 year before they give up the ghost, and those make perfectly AOK drives. Otherwise, if you're going to buy drive specifically for offline storage, check [DiskPrices](https://diskprices.com/) for the best dollar-per-terrabyte price possible. 5600 or 7200rpm, air or helium, 128 or 256mb cache: none of this matters when they're sitting in a drawer somewhere.


### Storing your drives {#storing-your-drives}

This bit might sound pedantic, but storing your drives properly is, in many ways, just as important as checking their S.M.A.R.T. status. I personally, keep my drives in Orico cases, labeled with their model number, capacity, and canonical name. The Orico cases claim to be shock and static resistent, but I still keep the drive in an antistatic bag with a small silica packet inside the case just to be safe. Stack these somewhere dry with little temperature fluctation. Harddrives are pretty hearty, but I'd rather be safe than sorry.


## Finding your flow. {#finding-your-flow-dot}

This is where things get optinionated and taylored to your needs. Personally I wanted a system that could:

-   be fully administered (backup, recovery, and maintenance) in a Linux shell -- bonus points if it's a live install with basic kernel primitives.
-   leave the files fully intact and accessable without any special software or de-serialization
-   locate the file among all the offline drives
-   validate the content of every block with checksums
-   replicate te files easily and implicitly

I chose `btrfs` and `git-annex`.

The filesystem was between btrfs and zfs, but I personally have personal experience with the former and dislike the experience of exporting and offlining ZFS pools. BTRFS is now include in the Linux kernel and has all the features I look for in a modern filesystem. Relevant to this use case: block-level deduplication, disk defragmenting, and data scrubbing.

Git Annex surprised me, honestly. I hadn't given it much thought in the past, but it cover my requirements fully and it, at the very least, aligns with my normal software development workflow. From their website:

> git-annex allows managing large files with git, without storing the file contents in git. It can sync, backup, and archive your data, offline and online. Checksums and encryption keep your data safe and secure. Bring the power and distributed nature of git to bear on your large files with git-annex.

Annex supports quite a few remote repository backends like web, bittorrents, xmpp, and S3 to name a few. Unless I decide to move files into AWS S3 or Glacier in the future, I'll only ever use the bare filesystem. I'd recommend at least reading through their [docs](https://git-annex.branchable.com/) -- they're wonderful!


## In practice {#in-practice}

First thing's first, we need to create an annex repository. I always keep one local repository on my system so I can run a quick `git annex whereis` when I need to locate a file.

```bash
mkdir -p ~/anenx && cd ~/annex

git init

git annex init
git annex numcopies 2
```

Once we've setup the \\(origin\\) remote, so to speak, we can format our drive, clone a copy of the annex, and create an "offline repo".

```bash
mkfs.btrfs

mkdir -p /mnt/drive0

mount /dev/sdb /mnt/drive0 && cd /mnt/drive0

git clone ~/annex

cd ~/annex
git annex group drive0 archive
git annex wanted drive0 standard
git annex describe drive0 "hdd-model-number"
```

At this point, we're ready to start backing up our files! I'll use rsync to copy files to the drive and add them to the annex. In this instance, lets backup an extremely value video files.

```bash
cd /mnt/drive0/annex
rsync -h --progress ~/Documents/big_buck_bunny.mp4 ./

git annex add ./big_buck_bummy.mp4
git commit -S -m "🅱️ig 🅱️uck 🅱️unny"
git annex sync
```

Syncing the file is a bit like pushing. It makes sure that all other remotes are aware. If we jump back to the origin repo and sync, we should be able to see the newly committed file. What's more, since we set the desired number of copies to 2, we can list all files missing a copy. We can even check the location of the file and grab some info on the drive -- all the info you'd ever need when recovering offline data.

```bash
cd ~/annex
git annex sync

git annex list

git annex list --lackingcopies 1

git annex whereis big_buck_bunny.mp4

git annex info drive0
```

Let's say this drive has the capacity for exactly one copy of `big_buck_bunny.mp4`. Before packing the drive away, I'll run a quick (buyer beware, this actually takes a while) defragmentation and data scrub to validate the file's integrity. This is likely overkill, but what about this process isn't?

```bash
btrfs filesystem defrag -vf /mnt/drive0
btrfs scrub /mnt/drive0
```

An important note: I'll pull this drive out every 4-6 months and run another scrub! Part of offline storge is actually spinning the disk up, mounting it, and making sure all the files are as you expect.
