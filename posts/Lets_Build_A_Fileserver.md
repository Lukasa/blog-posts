## Let's Build A Fileserver!

Those of you who know me will be familiar with the fact that I am what could
be called a 'digital packrat'. After a few years of university my DVD/Blu-Ray
collection has expanded to the point that it fills multiple shelves, and I
have digitised the whole thing.

This has turned into a more than 1.3 TB media collection, which has outgrown
the 1.5 TB drive it has been stored on. For this reason, I need a more
effective storage solution. This solution needs to be sustainable: I can't
keep copying the data onto ever-larger single hard drives. It also needs to
be safe: I can't rely on the occasional backup to another similarly-sized hard
drive in another machine.

Thus, the decision was made: have a dedicated fileserver. It needs to be
easily expandable, it needs to have data integrity built in, and it needs to
be cheap. I've finally gotten around to sorting this out, and have documented
the construction for posterity.

Alright, enough chatter. Let's get to it!

### The Build

Let's begin. First, obtain your fileserver. I got an
[HP Proliant Microserver](http://www.ebuyer.com/281915-hp-proliant-turion-ii-n40l-microserver-100-cashback-658553-421)
from Ebuyer, which came with a £110 cashback meaning I got it dirt cheap. It
has all the things you need from a fileserver: easy-access and plentiful drive
bays, quiet fans and enough RAM to run a modern filesystem.

Next, obtain your disks. This is actually the part that requires the most
thought. You'll see later in this post that we will be using a filesystem that
allows us to replace disks to increase the amount of space available to us,
so we should plan for the short-term, with the awareness that we can expand in
the long-term. For that reason, I got two
[2TB WD Caviar Green](http://www.ebuyer.com/264274-wd-2tb-green-desktop-drive-wd20earx)
drives. You can buy any drives that suit you, but bear in mind that you'll be
running this server for long periods of time, so you will probably want to get
energy-efficient drives as much as possible.

Note also that you will need a drive for the operating system. You could mount
a drive in the CD bay, or do what I did and use a USB stick to run the OS.

#### Step 1: Prepare The Server

We first need to make the server into a condition where we can install the
OS onto our system drive. I'm using a USB stick for the OS, which means I want
that to be the only disk in the server on first boot. To that end, open up the
server and remove any other hard drives inside it, leaving only the disk you
want the OS installed to.

Don't forget to hook up a monitor, a mouse and a keyboard to your server. It
is probably possible to do a command-line install of Ubtuntu, but I really
can't be bothered to try, so it'll be a graphical install.

Seal up your server, and plug your install USB into one of the ports on the
front. You're ready to go!

#### Step 2: Installation

Deep breaths now, and press the power button. You'll immediately see a lot of
pretty lights light up on the server, which is a good sign. If that doesn't
happen, then...crap. Happily, it worked for me, so I'm just going to assume
it worked for you.

Watch the POST and the prompts. Eventually you'll see a prompt offering you
the ability to press F10 to get to the BIOS: do that. When you're in the BIOS
make sure that the system time is right, because it wasn't for me. Make sure
of the date, too, although you should note that mine was set in US date/time,
so was right although it appeared to be wrong.

DO NOT CHANGE THE BOOT ORDER. If you have only two disks in, and only one of
them has an OS installed on it, you won't have a problem booting. Save and
exit the BIOS, and then allow the machine to boot. Hopefully, you'll end up
booting into a live-installation of Ubuntu.

You'll be asked if you want to 'Try Ubuntu' or 'Install Ubuntu'. We need some
of the utilities, so initially go to 'Try Ubuntu'.

The first thing we need to do is make sure our system disk is ready to take
an OS. To do this, go to the Dash Home (the top icon in the dock to the left)
and search 'partition'. This should give you GParted: launch it.

Initially, GParted should be showing you the partitions of your boot USB. To
show your future boot disk, click on the drop-down box in the top right and
select the other disk (in my case, `/dev/sdb`). If you're using a new HDD, you
should find it is entirelly unpartitioned space, which is great. If you're
using an off-the-shelf USB stick like me, you'll probably find that the
manufacturer has dumped some crap on your drive.

If your drive has any partitions, unmount them by right-clicking on them and
selecting 'Unmount'. Then, remove the partition by right-clicking on it and
selecting 'Delete'. Your device should then become entirely 'Unallocated'.
Click the green tick, and then 'Apply'. Then close GParted.

Now, double-click the Install icon on the desktop. It's time to go through the
install process. Follow the following steps:

- Select your language. Click 'Continue'.
- If you have cabled internet access, select 'Download Updates While
  Installing'. Definitely select 'Install Third-Party Software'. Select
  'Continue'.
- Select 'Erase disk and install Ubuntu', then 'Continue'.
- **IMPORTANT**. Make sure you have the correct drive selected. In my case,
  `sdb`. Then, click 'Install Now'.
- As the install proceeds, fill out the screens. Eventually, you'll have
  filled everything out. At this point, go make a cup of tea, and tweet about
  how much fun you're having. You should even send me a tweet! You could also
  do what I did, and put on some basketball (Let's Go Opals!).

The install will take a while (I watched a half of basketball during the
install), particularly if you're installing to a USB stick and/or downloading
packages. Eventually, you'll be asked to restart. Do that, and when the prompt
tells you to remove your install USB stick and let the restart run.

Once you've restarted, we want to make sure all of the system software is
up to date. Open the Dash Home and type 'update', then select the Update
Manager. Check for updates, then install everything. This, too, will take a
little while. Or, you know, a long while.

Eventually you'll need to restart. Do it.

#### Step 3: Prepare for ZFS.

One of the major reasons I had for using Ubuntu was because it has excellent
support for [ZFS](http://en.wikipedia.org/wiki/ZFS). Before we get around to
installing our storage drives, we should prepare our machine for use with
them.

Installing ZFS on Ubuntu is easy. We just want to add the [ZFS-PPA](https://launchpad.net/~zfs-native/+archive/stable)
to our list of software repositories. To do that, open a shell and type the
following commands:

    $ sudo add-apt-repository ppa:zfs-native/stable
    $ sudo apt-get update
    $ sudo apt-get install ubuntu-zfs

This will, unfortunately, also take a while. Another caffeinated beverage for
you, and maybe an episode of [Community](http://en.wikipedia.org/wiki/Community_(TV_series))
 (Six seasons and a movie!).

In the meantime, place your storage drives into two of the drive brackets. One
of your drives should go in place of the 250GB drive. This is because it is
currently not possible to replace a drive in a storage pool with a larger
capacity one, but it is possible to extend a pool. Thus, the short term loss
is a long-term gain.

Put these two brackets back into your server. Ideally, they should be mounted
as far as part as possible: hard drives *do* produce heat, and leaving them
apart will keep the case cooler.

With that done, boot your machine. While it boots, sit and idly ponder what
you will want to call your drive pool. For the purposes of this guide, it
will be called the intensely-boring `dpool`.

#### Step 4: Set Up Your Drive Pool.

Once the machine has rebooted, get yourself another terminal. You'll want to
work out what the drive ids of your two new drives are. To do this, run:

    $ ls -l /dev/disk/by-id/

You'll see a whole lot of printouts. The filenames, highlighted in blue, are
the IDs associated with various disks. To work out which are the two new disks
you added, you want to consider their symlinks, which are shown in yellow.

Your two new drives will have three IDs associated with them, all symlinked to
the same place: one beginning `ata-`, one beginning `scsi-SATA` and one that
is shorter and manufacturer specific. Your new drives should have links that
point to `../../sd[a-z]`. To put it another way, their symlinks shouldn't end
in a number. On my machine, I see symlinks for `sda`, `sdb`, `sdc`, `sdc1`,
`sdc2` and `sdc5`. From that, I can tell that `sdc` is my boot disk, as it's
the only drive with partitions on it. Thus, `sda` and `sdb` are the two disks
I added. Take a note of their IDs: I used the ones beginning with `scsi-SATA`
but I know of no reason you couldn't use the others.

Now, create your first drive pool. Run the following command, replacing
`dpool` with the name of your choice.

    $ sudo zpool create dpool mirror /dev/disk/by-id/[id_of_first_disk] /dev/disk/by-id/[id_of_second_disk]

This command, unlike so many others, will run fast. You'll notice that, all
of a sudden, `/dpool` exists. (To see this, run `ls /`). Ta-da! You have
created your first software RAID. The eagle-eyed amongst you will note that I
have created a RAID-1: that is, mirrored storage. This is deliberate.
You cannot add disks to a RAIDZ, which means I do not gain the space advantage
unless I have four disks in place from the start. That seems financially silly
to me, so I'll instead use two disks as a mirror for now, and add another pair
when I need them.

From here, you can create sub-storage locations managed by ZFS using the
`zfs create` command. For instance, if you (like me) want to store movies, you
could run `sudo zfs create dpool/movies`. If you now `ls /dpool` you'll see
your new location exists.

To take a look at your new drive pool, run `sudo zpool status`. To see your
sub-sections, run `sudo zfs list`.

#### Step 5: Sharing.

Cool, so, I'm sick of sitting by this server now, so let's remotely
administrate it so I can lie down on the sofa watching the world's best
athletes tire themselves out.

I'm a reasonably security-conscious guy, so I don't just want anyone to be
able to log into my server as my user account, which will be the designated
administrator. For that reason, I'm going to end up using certificate-based
SSH, rather than password-interactive login. First, though, let's make sure
we can login as me at all.

Let's install OpenSSH:

    $ sudo apt-get install openssh-server

Once that takes, open up `/etc/ssh/sshd_config` in your favourite text editor
(which should be Vim). Turn off `X11Forwarding` (you'll never use it), then
save it and see if you can log in. With good luck, you'll be able to.

Brilliant. Next, you need a private and a public SSH key. If you already have
one, great, if not, on your remote machine run:

    $ cd ~/.ssh
    $ ssh-keygen -t rsa

Use no passphrase and the default filename.

Now you should have a certificate. To use it, first use your SSH connection to
your server to run `mkdir ~/.ssh`. Then, run:

    $ scp ~/.ssh/id_rsa.pub <user>@<server_ip>:.ssh/authorized_keys

Awesome. Now, disable the ability to use the keyboard to login entirely.
Reopen `sshd_config`. Uncomment `PasswordAuthentication` and change it to No.
These changes will take effect next time you restart.

Now, we want to set up file shares. The easiest way to share the files you
want to as many people as possible is to use SMB, which on Linux means using
Samba. Let's install it:

    $ sudo apt-get install samba

When that install finishes, open up `/etc/samba/smb.conf`. Uncomment
`security = user`. Then, scroll to the bottom of the file and add a new share.
This should be a block of text that looks a bit like this:

    [share]
        comment = An example share.
        path = /dpool/movies
        browsable = yes
        guest ok = yes
        read only = no

Save the file and exit it. Then, restart Samba:

     $ sudo restart smbd
     $ sudo restart nmbd

Awesome. You can now get access to your files.

#### Step 6: Transfer your files.

If you have any files sitting anywhere that you want to put on your server,
start doing it now.

#### Step 7: File Integrity.

One of the many advantages of using ZFS is that it provides a great deal of
data integrity. However, it's important to note that this does not entirely
come for free. Data integrity issues caused during transfers or copy-on-write
can be caught automatically by the filesystem, but there can be other forms of
data corruption that ZFS cannot automatically catch.

For that reason, it is important to scrub your filesystem regularly. As far as
I can tell the best practice is to scrub the filesystem [weekly](http://www.solarisinternals.com/wiki/index.php/ZFS_Best_Practices_Guide)
when using consumer-grade drives like ours.

Additionally, if a disk fails ZFS will limp onwards rather than falling over
and dying. This means we want to check on a regular basis to find out if any
of our disks have failed, so that we can intervene and replace the drive.

Doing this manually is boring, rote work. More importantly, it's also a job
that is easy to forget to do. For that reason, we'll write up a couple of
scripts to notify us instead. Also because, you know, scripts are cool.

We need two cron jobs. The first can be simply and easily added to your
`crontab` file. Open up `/etc/crontab` and add the line:

    0 0 * * 0   root    zpool scrub dpool

For those of you that don't speak `cron`, that means that once a week, at
midnight on Sunday, the command `zpool scrub dpool` will run. You don't need
to use `sudo` to elevate priveleges, as this command will run as root.

The status test needs to be scripted, because we want to be emailed. In fact,
we'll be using two scripts, because we need to be able to send an email if
something goes wrong. For that reason, we'll be using a shell script and a 
Python script, because Python is brilliant.

Firstly, let's write a Python script that can email you the content of a file.
[Here](https://gist.github.com/3277447) is one I whipped up in about five
minutes. If you want to use this yourself, you'll need to set your constants
to be correct at the top of the file.

Next, we want to write a quick bash script that will check the state of the
filesystem and email you the report if something looks amiss. My first pass
at this script looks [like this](https://gist.github.com/3277961).

When you've written these two scripts, put these two files into
`/usr/local/bin` and make sure they're executable. Then, we add the following
line to our crontab:

    10 8 * * *   root    zfs_report

At ten minutes past 8 in the morning, every day, this script will run.
Hopefully, this will be enough to alert you when a drive fails. Still, from
time-to-time a manual test will probably help you sleep more easily at night.

#### Step 8: Finalising.

Hurrah! You now have a fully-functioning fileserver. Hopefully you have all of
your data transferred across, and file sharing set up. Now is the time to make
sure you'll be comfortable administrating it. I copied across some of my
dotfiles, including vim themes and my `.vimrc`. I also installed `zsh`, and
made sure to use [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh/) and
my custom `zsh` theme.

The future will include the addition of another `vdev` to the drive pool,
which will probably be another pair of mirrorred drives. I'll probably have
another small blog post when I add drives to this server. For now, I'll go
back to watching the Olympics.
