# Disclaimer
A simple modification from xfs_undelete for salvaging files from broken xfs images.
Check up the inode number of the salvaged file before use. ABSOLUTELY NO WARRANTY.

Usage:
```sh
$ ./xfs_undelete (-s inode_number_for_salvage) broken_drive_imagefile.img
```
Recommend: when you use this tool for [xfs_salvage](https://github.com/shuheikurita/xfs_salvage), it is recommended to backup the whole disk with `ddrescue`:
```sh
$ sudo ddrescue -d /dev/BROKEN_DRIVE broken_drive_imagefile.img DRIVE.fullimage.log
```

# xfs_undelete
An undelete tool for the XFS filesystem.

## What does it?
*xfs_undelete* tries to recover all files on an XFS filesystem marked as deleted.
You may also specify a date or age since deletion, and file types to ignore or to recover exclusively.

*xfs_undelete* does some sanity checks on the files to be recovered.
This is done to avoid recovering bogus petabyte sized sparse files.
In addition, it does not recover anything unidentifiable (given you have the *file* utility installed) by default.
Specify *-i ""* on the command line if you want to recover those unidentifiable files.

The recovered file is stored on another filesystem in a subdirectory, by default *xfs_undeleted* relative to the current directory.
The filename cannot be recovered and thus, it is put as the time of deletion, the inode number, and a guessed file extension.
You have to check the recovered files you are interested in by hand and rename them properly.

## How does it work?
*xfs_undelete* traverses the inode B+trees of each allocation group, and checks the filesystem blocks holding inodes for the magic string *IN\0\0* that indicates a deleted inode.
Then, it tries to make sense of the extents stored in the inode (which XFS does not delete) and collect the data blocks of the file.

## Is it safe to use?
Given it only ever *reads* from the filesystem it operates on, yes.
It also remounts the filesystem read-only on startup by default so you don’t accidentally overwrite source data.
However, I don’t offer any warranty or liability. **Use at your own risk.**

## Prerequisites
*xfs_undelete* is a tiny Tcl script so it needs a Tcl interpreter.
It makes use of some features of Tcl-8.6, so you need at least that version.
The *tcllib* package is used for parsing the command line.
It also needs a version of *dd* which supports the *bs=*, *skip=*, *seek=*, *count=*, *conv=notrunc*, and *status=none* options, as well as a *readlink* which supports the *-e* option, and a version of *stat* which supports the *-L* and *--format=%m* options.
The ones from GNU core utilities will do.
If the *file* utility and magic number files with MIME type support are installed (likely), *xfs_undelete* will use that to guess a file extension from the content of the recovered file. In short:

- tcl >= 8.6
- tcllib
- GNU coreutils

Recommended:

- file (having magic number files with MIME type support)

In addition, you need enough space on another filesystem to store all the recovered files as they cannot be recovered in place.

## Distribution Packages
[<img src="https://de.opensuse.org/images/1/10/Opensuse-geeko.png" alt="OpenSUSE Linux" height="100px">](https://software.opensuse.org/package/xfs_undelete)
[<img src="https://www.archlinux.org/static/logos/archlinux-logo-dark-90dpi.ebdee92a15b3.png" alt="Arch Linux" height="100px">](https://aur.archlinux.org/packages/xfs_undelete/)

## Limitations
- The way XFS deletes files makes it impossible to recover the filename or the path. You cannot undelete only certain files. The tool however has a mechanism only to recover files deleted or modified since a certain date. See the -t and -T options.
- The way XFS deletes files makes it impossible to recover heavily fragmented files. For typical 512 byte inodes, you can only recover files having at maximum 21 extents (of arbitrary size). Files with more extents cannot be recovered at all by this program.
- The way XFS deletes files makes it impossible to retrieve the correct file size. Most files will be padded with zeroes so they fit the XFS block size. Most programs do not bother anyway. Files of the text/ mimetypes get their trailing zeroes trimmed by default after recovery. See the -z option to change this behaviour.

## License
*xfs_undelete* is free software, written and copyrighted by
Jan Kandziora &lt;jjj@gmx.de&gt;. You may use, distribute and modify it under the
terms of the attached GPLv3 license. See the file LICENSE for details.

## How to use it

There's a manpage. Here is a copy of it:

# NAME

xfs\_undelete - an undelete tool for the XFS filesystem

# SYNOPSIS

**xfs\_undelete** \[ **-t** *timerange* \] \[ **-T** *timerange* \] \[
**-r** *filetypes* \] \[ **-i** *filetypes* \] \[ **-z** *filetypes* \]
\[ **-o** *output\_directory* \] \[ **-s** *start\_inode* \] \[ **-m**
*magicfiles* \] \[ **--no-remount-readonly** \] *device*  
**xfs\_undelete -l** \[ **-m** *magicfiles* \]

# DESCRIPTION

**xfs\_undelete** tries to recover all files on an XFS filesystem marked
as deleted. The filesystem is specified using the *device* argument
which should be the device name of the disk partition or volume
containing the filesystem.

You may also specify a date or age since deletion, and file types to
ignore or to recover exclusively.

The recovered file cannot be undeleted in place and thus, it is stored
on another filesystem in a subdirectory, by default *xfs\_undeleted*
relative to the current directory. The filename cannot be recovered and
thus, it is put as the time of deletion, the inode number, and a guessed
file extension. You have to check the recovered files you are interested
in by hand and rename them properly. Also, the file length cannot be
recovered and thus, the recovered files are padded with **\0**
characters up to the next xfs block size boundary. Most programs simply
ignore those **\0** characters but you may want to remove them by hand
or automatically with the help of the **-z** option.

This tool does some sanity checks on the files to be recovered. That is
to avoid "recovering" bogus petabyte sized sparse files. In addition, it
does not recover anything unidentifiable (given you have the file
utility installed) by default. Specify **-i** *""* on the command line
if you want to recover those non-bogus but still unidentifiable files.

# OPTIONS

**-t** *timerange*  
Only recover files that have been deleted within the given time range.
The *timerange* value has to be put either as two timespecs separated by
a doubledot e.g. *2020-03-19..-2hours*, as a doubledot followed by a
timespec as e.g. *..-2hours*, which means a range starting at epoch, as
a timespec followed by a doubledot as e.g. *-2hours..*, which means a
range ending now, or as a single timespec value. The latter means the
same as a timespec followed by a double dot. Timespecs may be all values
Tcl's \[clock scan\] function accepts. See **clock**(n). By default,
files deleted from epoch to now are being recovered.

**-T** *timerange*  
Only recover files that have been modified within the given time range
before they have been deleted. This option is useful if you know the
date of your latest backup. The *timerange* value has to be put either
as two timespecs separated by a doubledot e.g. *2020-03-19..-2hours*, as
a doubledot followed by a timespec as e.g. *..-2hours*, which means a
range starting at epoch, as a timespec followed by a doubledot as e.g.
*-2hours..*, which means a range ending now, or as a single timespec
value. The latter means the same as a timespec followed by a double dot.
Timespecs may be all values Tcl's \[clock scan\] function accepts. See
**clock**(n). By default, files modified from epoch to now are being
recovered.

**-r** *filetypes*  
Only recover files with a filetype matching a pattern from this
**comma**-separated list of patterns. See section **FILETYPES** below.
By default this pattern is \* ; all files are being recovered, but also
see the **-i** option.

**-i** *filetypes*  
Ignore files with a filetype matching a pattern from this
**comma**-separated list of patterns. See section **FILETYPES** below.
By default this list is set to *bin* ; all files of unknown type are
being ignored, but also see the **-r** option.

**-z** *filetypes*  
Remove trailing zeroes from files with a filetype matching a pattern
from this **comma**-separated list of patterns. See section
**FILETYPES** below. By default this list is set to *text/\** ; all
files of text/\* mimetype have their trailing zeroes removed.

**-o** *output\_directory*  
Specify the directory the recovered files are copied to. By default this
is *xfs\_undeleted* relative to the current directory.

**-s** *start\_inode*  
Specify the inode number the recovery should be started at. This must be
an existing inode number in the source filesystem, as the inode trees
are traversed until this particular number is found. This option may be
used to pickup a previously interrupted recovery. By default, the
recovery is started with the first inode existing.

**-m** *magicfiles*  
Specify an alternate list of files and directories containing magic.
This can be a single item, or a **colon**-separated list. If a compiled
magic file is found alongside a file or directory, it will be used
instead. This option is passed to the **file** utility in verbatim if
specified.

**--no-remount-readonly**  
This is a convenience option meant for the case you need to recover
files from your root filesystem, which you cannot umount or remount
read-only at the time you want to run *xfs\_undelete*. The sane solution
would be moving the harddisk with that particular file system to another
computer where it isn't needed for operation.

If you refuse to be that sane, you have to make sure the filesystem was
umounted or remounted read-only at least in the meantime by another
means, for example by doing a reboot. Otherwise you won't be able to
recover recently deleted files.

**USE THIS OPTION AT YOUR OWN RISK.** As the source filesystem isn't
remounted read-only when you specify this option, you may accidentally
overwrite your source filesystem with the recovered files.
*Xfs\_undelete* checks if you accidentally specified your output
directory within the mount hierarchy of your source filesystem and
refuses to do such nonsense. However, automatic checks may fail, so
better check your specification of the output directory by hand. Twice.
It **must** reside on a different filesystem.

**-l**  
Shows a list of filetypes suitable for use with the **-r**, **-i**, and
**-z** options, along with common name as put by the **file** utility.

# FILETYPES

The *filetypes* as used with the **-r**, **-i**, and **-z** options are
a **comma**-separated list of patterns. Patterns of the form \*/\* are
matched against known mimetypes, all others are matched against known
file extensions. The file extensions are guessed from the file contents
with the help of the **file** utility, so they don't neccessarily are
the same the file had before deletion.

Start *xfs\_undeleted* with the **-l** option to get a list of valid
file types.

**Note:** you want to quote the list of filetypes to avoid the shell
doing wildcard expansion.

# EXAMPLES

\# cd ~ ; xfs\_undelete /dev/mapper/cr\_data

This stores the recovered files from /dev/mapper/cr\_data in the
directory ~/xfs\_undeleted.

\# xfs\_undelete -o /mnt/external\_harddisk /dev/sda3

This stores the recovered files from /dev/sda3 in the directory
/mnt/external\_harddisk.

\# xfs\_undelete -t 2020-03-19 /dev/sda3

This ignores files deleted before March 19th, 2020.

\# xfs\_undelete -t -1hour /dev/sda3

This ignores files deleted more than one hour ago. The -t option accepts
all dates understood by Tcl’s \[clock scan\] command.

\# xfs\_undelete -i "" -t -2hour /dev/sda3

This recovers all files deleted not more than two hours ago, including
"bin" files.

\# xfs\_undelete -r 'image/\*,gimp-\*' /dev/sda3

This only recovers files matching any image/ mimetype plus those getting
assigned an extension starting with gimp-.

# TROUBLESHOOTING

When operating on devices, this program must be run as root, as it
remounts the source filesystem read-only to put it into a consistent
state. This remount may fail if the filesystem is busy e.g. because it's
your */home* or */* filesystem and there are programs having files
opened in read-write mode on it. Stop those programs e.g. by running
*fuser -m /home* or ultimately, put your computer into single-user mode
to have them stopped by init. If you need to recover files from your /
filesystem, you may want to reboot, then use the
**--no-remount-readonly** option, but the sane option is to boot from a
different root filesystem instead, for example by connecting the
harddisk with the valueable deleted files to another computer.

You also need some space on another filesystem to put the recovered
files onto as they cannot be recovered in place. If your computer only
has one huge xfs filesystem, you need to connect external storage.

If the recovered files have no file extensions, or if the **-r**,
**-i**, and **-z** options aren't functional, check with the **-l**
option if the **file** utility functions as intended. If the returned
list is very short, the **file** utility is most likely not installed or
the magic files for the **file** utility, often shipped extra in a
package named *file-magic* are missing, or they don't feature mimetypes.

# SEE ALSO

**xfs**(5), **fuser**(1), **clock**(n), **file**(1)

# AUTHORS

Jan Kandziora &lt;jjj@gmx.de&gt;
