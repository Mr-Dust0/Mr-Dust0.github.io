---
title: FAT
date: 2024-03-04
published: true
categories: ["File System"]
tags: ["FAT", "Forensics"]
---
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-PDJMTCGM1Q"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-PDJMTCGM1Q');
</script>

# FAT (file allocation table)
The file allocation table (FAT) file system was originally developed in 1977 for use on floppy disks and the windows 9x operating systems. However,
it is still in use today in removable storage devices like usbs this is because fat has a relatively small overhead and 
is supported by all the operating systems, so it is a good medium to move around data. Also, Fat is simple because it only 
really relies on two data structures the file allocation table and the directory entries which reduces the size and complexity
needed to support it in the operating system kernel.
## General
The basic idea of a FAT file system is that each file and directory is allocated a data structure called a directory entry
(32 byte in length) that contains the starting address, file name, size, file content and other pieces of metadata. The data
in the FAT file system is stored in a data unit called a cluster (like ntfs) if the data in a file or directory needs more than one cluster
to contain all the data the other clusters needed to store the data are found in the file allocation table. This file allocation data structure is used to identify 
the next cluster in a file or directory and is also used to identity the allocation status of the cluster (if it is already being
used or not).

![img_11.png](/assets/img/FAT/img_11.png)


In the example above the name.png points to the 4th cluster which means we look at the 4th entry in the fat to see if there are any more 
clusters needed for the file. The 4th entry in the points to the fat entry 10 which means that the file always uses cluster
10 to store data and that points to 1st entry and then the first entry points to the 3rd entry which is negative one which is
a end of file (eof) marker. So we know that name.png uses clusters 4,10,1,3 to store its data.

##  Directory entry data structure
The directory entry is a data structure that is allocated for every file and directory. It is 32 bytes in size and contains the file's attributes, size, starting cluster, and dates and times.


![img_12.png](/assets/img/FAT/img_12.png)


The image above is the structure of a directory entry in FAT. The first byte in the directory entry can show if the file or directory has been deleted
which can help searching for files that have been deleted but not yet overwritten. Also, can see what data the file used by looking
at bytes 26-27 to see the address of the first cluster and then follow the file allocation table to see what other clusters where being used 
by the file however if the cluster has been reallocated this can show a false presentation of the deleted file and make recovery of the original data nearly impossible.

## File allocation table data structure
There are normally two file allocation tables in a file system with one being for backup if the original file allocation table
gets corrupted but the exact number is given in the boot sector the file allocation table. The total size of each File allocation table
is given in the boot sector.The table consists of equal-sized entries and has no header or footer values which can make it more
difficult to search in file systems because it has no magic values. The size of each entry depends on the file system version
with FAT12 having 12 bit entries, FAT16 having 16 bit entries and FAT32 having 32 bit entries. The entries of the File allocation
table start from zero with each entry's address corresponding to the cluster with the same address. Entry's with 0 in them mean
they are not allocated to a file if an Entry is allocated it will be a non-zero and will the address of the next cluster in the file or
directory. The last cluster in the file corresponding file allocation table will contain an end of file value which is a large
value like 0xffff.
## FAT Maximum sizes
The maximum cluster size in a FAT file system is 64KB. And because in FAT12 only 12 bit are used for
the entry in the File allocation table which means the maximum address is 2<sup>12</sup> = 4096 and 12 clusters are reversed
for things like boot code and the file allocation itself the maximum size of FAT12 file system is 4084 * 64kb which is around
256MB which is nothing in today's world which shows why the FAT file system was replaced by the NTFS file system which can 
support larger file systems and has more advanced features like more attributes and more logging. The maximum size of an FAT16 filesystem is FAT16: 4GB (for 64kB clusters) because it 
has 4 more bits for addresses. Even if the size was good the fact that clusters have to be 64kb means that the space won't
be used up efficiently and there will be alot of wasted space in the file system.
The FAT32 has a maximum of 2TB (512B sectors) which is alot better but still is not big enough and FAT doesn't support advanced 
features like journaling which means that it is not used as a primary computer file system in today's world.
