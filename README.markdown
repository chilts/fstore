fstore - A tool to store, sync, backup and label any kind of file.

# Concepts and Terms

FStore has some ideas about certain cencepts and term which will be detailed
here. Even though this might be quite a full-on introduction, it means things
will become clearer faster.

## The 'Hash' Store

In the top level directory of your storage area, FStore creates and uses a
hidden directory called .fstore to save both metadata and file data related to
the files you wish to store. This has the following two subdirs: 'metadata' and
'hash'. It also contains a file called 'config'.

To set up a new storage area for FStore, cd into that directory and type the
following command:

    $ cd path/to/store
    $ fstore init

This will create the relevant directories and set up fstore for immediate use.

## Metadata

Each and every file stored with fstore has a corresponding metadata file named
after it's MD5 and stored inside `.fstore/metadata/$md5`. It's format is:

    filename <filename>
    tags <category>:<label>...

## Tags

For every file saved to FStore, it _must_ be given at least one TAG. And each
TAG consists of a CATEGORY:LABEL combination. This essentially gives the
2-level directory hierarchy whereby the files can eventually be found. The
files are stored in .fstore/hash/ as their MD5 hash, but a symlink will be made
from $category/$tag/$filename to the hashed file.

For example, the following command puts the song.mp3 file into the 'MyBand'
directory:

    $ fstore add -t MyBand:MyAlbum song.mp3

This would perform the following actions:

* mkdir -p MyBand
* md5sum song.mp3 # 3c5d639d1a90f7ea598230276c5b2485
* cp song.mp3 .fstore/hash/3c5d639d1a90f7ea598230276c5b2485
* ln -s .fstore/hash/3c5d639d1a90f7ea598230276c5b2485 MyBand/MyAlbum/song.mp3

And the following puts all of the JPGs found into the 'Holiday/Crete/'
directory:

    $ fstore add -t Holiday:Crete *.jpg

Similar actions to above would happen this time.

When adding labels, very similar processes occur except a 'label' directory is
create within the existing ./labels/ directory and the symlinks to the originl
file are created there.

# Commands

## 'add'

To add a new file to FStore, use the following command:

    $ fstore add --tag... path/to/file...

Note: each tag should be in the format <category>:<label>. 

This will add all of the files specified to FStore. By doing this it will do
three things:

* take an MD5 of the file
* move the file into .fstore/hash/ and save it under the name of the MD5 hash
* loop through all the tags for the following commands:
** create a symlink from category/label/originalfilename to the hashed file
** save a TXT file (also called the MD5) into .fstore/metadata/ which contains
   various bits of information about the file (original filename and any labels)

## 'sync'

ToDo: ...

## 'move'

Files can be moved from one directory to another:

    $ fstore move directory/filename new-dir/
    $ fstore move directory/filename new-dir/new-filename

This moves the file from one directory to another and optionally renames the
file. In this case the MD5 and all labels stay the same.

## 'rename'

You can rename a file within FStore by issuing the following command:

    $ fstore rename directory/filename new-filename

This renames a file within it's current directory. It's MD5 and all labels stay
intact.

## 'remove'

To remove a file from FStore issue the following command:

    $ fstore remove directory/filename

In this case, the file that exists within the directory is left on the
filesystem.

## Deleting a File Permanently

To remove a file from FStore and remove it permanently, issue the following:

    $ fstore delete directory/filename

The file and any associated symlinks as well as it's metadata is removed. If
the directory no longer contains any other file, it is also deleted.

## 'tag' and 'untag'

Tags can be added or removed from each file by issuing the following commands:

    $ fstore tag -t category/label <file...>
    $ fstore untag -t category:label <file...>

The above commands add or remove a label from the file. Symlinks inside label
directories are also added/removed and the metadata information about the file
is also removed.

## 'reindex'

If you've gone and added a load of new files to your storage area (such as
GranniesAppleTree/{Spring,Summer,Autumn,Winter.jpg) you can get FStore to
automatically add those files to it's tracked files.

    $ fstore reindex

This will find all files in ./ and below which are untracked and add them in
the usual way, except again no labels are added. Files inside ./ are ignore
(since they don't have a directory and all files must reside in a directory).

## 'sync'

FStore can sync with other FStore either locally or on other machines. The
default process when syncing is to never delete any data, whether this is file
data, directories or labels.

However, you can also run the sync in 'interactive' mode, which asks what you'd
like to do at each stage of the process.

    $ fstore sync . /path/to/remote/
    $ fstore sync . host:/path/to/remote/

This will try and update the remote if it can, but will fatally fail if it
can't. It is not up to FStore to maintain any kind of filesystem or OS
permissions. It will generally use a plain 'ssh/scp' command to modify what it
can (by way of the Net::SSH and Net::SCP modules).

## 'fsck'

# Other Things to Think About

## Duplicate Files

Within FStore there is nothing stopping you adding the same file twice into
different directories. For example, you might like to add a photo of your Gran
(gran-by-the-fireside.jpg) into both the 'Christmas2010' directory and the
'Gran' directory. FStore can help you find these duplicates and can ask what
you'd like to do with them. Generally you'd like to delete them but we're not
going to prescribe what you'd like to do.

Alternatively, you may like to add that file to the Christmas2010 directory but
add the 'Gran' label to it. It's up to you whether you'd like to do one or
other of the above, with the added advantage that the file is only actually
stored on the filesystem once.

## What FStore _doesn't_ do

FStore doesn't:

* remember any history of any file
* 

FStore is useful:

* for files that never or infrequently change
* for files such as photos and MP3s
* for files that you'll never edit

Note: FStore can be used for _any_ file, it just might not be as useful nor
what you want.

## Culling Old Files

Sometimes you may end up with a file inside the store which doesn't have a related ...



Files are written and stored inside a hidden directory called '.store' and
symlinks to them are created in the current directory. So for example, let's
say you had a file called 'tree.jpg', you could add it as follows:

    $ fstore tree.jpg tree 20110427 easter-holiday

If the file had an MD5 of '9f09bdc8278555fe26ac984fca2f1562', the file would be
stored inside .fstore as 9f09bdc8278555fe26ac984fca2f1562.

The directory structure also created would be:

    $ ls -lR
    .:
    total 12
    drwxr-xr-x 2 andy andy 4096 2011-04-27 23:03 20110427
    drwxr-xr-x 2 andy andy 4096 2011-04-27 23:03 easter-holiday
    drwxr-xr-x 2 andy andy 4096 2011-04-27 23:03 tree

    ./20110427:
    total 0
    lrwxrwxrwx 1 andy andy 43 2011-04-27 23:03 hello.txt -> ../.fstore/9f09bdc8278555fe26ac984fca2f1562

    ./easter-holiday:
    total 0
    lrwxrwxrwx 1 andy andy 43 2011-04-27 23:03 hello.txt -> ../.fstore/9f09bdc8278555fe26ac984fca2f1562

    ./tree:
    total 0
    lrwxrwxrwx 1 andy andy 43 2011-04-27 23:03 hello.txt -> ../.fstore/9f09bdc8278555fe26ac984fca2f1562

So you can see that the three labels correspond to directories that have been
created, and a symlink file resides in each which points back to the contents
of that file. Note: any filename added to the fstore must be unique.

Also note: filenames which have the same MD5 are stored twice but will appear
as two lots of symlinks using the original filename they were added as.

-------------------------------------------------------------------------------
