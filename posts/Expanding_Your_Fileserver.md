## Expanding Your Fileserver

Quite some time ago, I wrote a blog post detailing how I made a
[fileserver](http://lukasa.co.uk/2012/08/Lets_Build_A_Fileserver/) to store my
media collection on. Well, time has passed and my media collection has
expanded, and I got to the point that my ZFS drive pool was more than 90% full.
This meant it was time to think about expanding the storage.

In my original post I promised that I would write a blog post when I expanded
the storage. As it turns out, doing this is pretty easy, so this blog post will
be fairly short.

The first thing I did was buy two more disks. The price of 3TB hard drives
hasn't greatly reduced since I wrote the last post, so it seemed like the right
thing to do was buy another two
[WD 2TB Green](http://www.ebuyer.com/264274-wd-2tb-green-desktop-drive-wd20earx)
drives.

Once the drives arrived, I slotted them into the spare two hard drive bays in
the chassis. This also involved moving the drives around so that the two drives
in each mirrored pair were _not_ adjacent to each other. This should ensure
that they don't wear evenly, and as a result one should fail well in advance of
the other. This is actually ideal, as it will buy us time to resilver a drive
if something goes horribly wrong.

When those two drives were plugged in, I turned the server back on and logged
in. Once done, I ran the following command:

    ls -l /dev/disk/by-id/ | grep scsi\-SATA

This showed all the SATA drives in the box, and includes their symlinks. I then
looked for the two drives that I had just inserted. I was able to identify them
by the fact that they do not have partitions on them. The two drives that had
been in from the start were had three associated symlinks: for example,
`../../sdc`, `../../sdc1` and `../../sdc9`. The two new drives turned out to be
`../../sdb` and `../../sdd`. I found their long names, which should be of the
form in the original blog post.

Once you've worked that out, adding new drives is trivial:

    zpool add <pool name> mirror /dev/disk/by-id/<disk1 id> /dev/disk/by-id/<disk2 id>

This should take a second, but then you'll be done! If you run `zpool status`
you should see your new mirror underneath your drive pool.

Enjoy your huge quantities of new space!