# heal-harddrive.sh

This script works on FreeBSD and repairs faulty sectors by
using the S.M.A.R.T. replacement mechanisms of your harddrive.
It only works on harddrives with 4k sectors (for now).

Be aware that running it on a drive containing a filesystem
will DESTROY data, metadata and perhaps even the entire filesystem.

**MAKE SURE YOU HAVE BACKUPS!**

## How it works

The command will run over entire drives or parts of it and check if there
are any I/O errors. If it finds any, it will overwrite one 4k sector with zeros
and then continue scanning from there on.

The operation is slow, because it reads single LBAs (512 bytes). If it finds
an error, the repair procedure takes about 30s per sector. It also makes
sure that a single sector read causes an error and aborts the entire procedure in
cases that are unexpected.

## Do not forget to repair filesystems!

This script destroys the filesystem. Yes, this is not a joke. It selectively
overwrites parts of the filesystem on the harddrive with zeros.

After running this script you will need to repair the filesystem, if there
was any (use `fsck` or `zpool scrub`).

## Why is this script needed at all

Most filesystems and device drivers do not enforce S.M.A.R.T. replacement.
This is especially annoying when you have data that is read often, but it
is not written again. The harddrive won't replace it, because it never
writes to the faulty sectors. You always get read errors. It gets even
more horrible on copy-on-write filesystems, where sectors are not reused
but allocated at other offsets. In this case you lose track of this
faulty sector that might appear later again.

## What you should expect

The script has not been run on many harddrives, yet. Please consider it
experimental and even worse, keep in mind that it **really** destroys data
to repair a harddrive.

## When to use this script

This script can be used when S.M.A.R.T. detect errors on a drive and there
are still replacement sectors available. Watch this S.M.A.R.T. attribute:

```
# smartctl -A /dev/ada0
.
.
.

197 Current_Pending_Sector  0x0032   200   200   000    Old_age   Always       -       213
.
.
.
```

It shows that there are `213` sectors to replace, but S.M.A.R.T. has not replaced
them, yet.

## When NOT to use this script

When you are **not sure** what this script does and when you are **not sure**
what the source of the errors is. A harddrive is a part of a complex piece of hardware.
There might be multiple problems that lead to misbehavior of a harddrive. It starts
by flaky cables, instable power sources, vibrations and much more.

DMA transfer errors are **clearly not** caused by faulty sectors.

## Usage examples

```
./heal-harddrive.sh /dev/ada0
```

Scan device `ada0` for errors and heal them.

```
./heal-harddrive.sh /dev/ada1 100000
```

Scan device `ada1` for errors starting from LBA `100000` and heal them.
