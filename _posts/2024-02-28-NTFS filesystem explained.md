---
title: MFT
date: 2024-02-28
published: true
categories: ["File System"]
tags: ["NTFS",  "MFT", "Forensics"]
---
# MFT
## General
A feature of NTFS that sets it apart from other file systems is that the entire file system is considered data area which
means that any sector can be allocated to a file.The only layout an NTFS file system has is that the first sectors of the volume
contain the boot sector and the boot code.
## MFT General
The master file table is the main feature of the NTFS file system because it contains information on all the files and 
directory's. Each file and directory has at least one entry in the MFT which is 1KB in size but can have more if the file 
or directory needs more attributes.The first 42 bytes of the MFT entry has a specific purpose, and the rest of the data is filled
up with attributes (these are small data structures that have a specific purpose like storing the file name).

The first entry in the MFT is always $MFT which describes the MFT file location on disk. This is the only place where the size
and location of the MFT is found so this needs to parse so the rest of the files and directory's can be located on the file system. The starting location of 
the MFT is described in the boot sector (which is the first sector of the filesystem) so the operating system can locate and parse the MFT when loaded.The size of the MFT entry is stored in the boot sector but has always been 1KB in all versions of windows 
(this is why if you create a file which very little data the size will still be a 1KB in size).  

## What are MFT entries
MFT entry's uses attributes to store every think about the file and directory and while each type of attribute stores a 
different type of data, all attributes have two parts: the header and the content. Attribute sizes can vary because attributes
can store things like the filename or the file content which may be MB or GB in size. Because it would be impossible to 
store large attributes into the MFT entry (which as to date has only been 1KB) there is two types of attributes an resident
attribute where the attribute is stored in the mft entry and a non-resident attribute where the data is stored in an 
external cluster run. The header of the attribute states weather the attribute is resident or not. If the attribute is resident
then the data will for the attribute will come after the header if it is non-resident then the header will give the cluster address. 
### Forensic Use
This means if doing a forensic analysis of an NTFS file system if the data attribute for the file is resident then the file is deleted
the data may be recoverable. Since when a file is deleted the mft entry isn't actually deleted it is just shown to be not in
use that other files can use it by overwriting the entry. So if the entry isn't overwritten by other file being created 
(the time taken for this to happen depends on how munch activity is done on the filesystem) then will still be able to find
the file content in the entry for the file which was deleted.
###  ADS ($Data Attribute)
An Alternative data streams is when a file has one than one $Data attribute which is the attribute that stores the files content.
The default $DATA attribute that is created when a file is created does not have a name associated with it, but additional $DATA attributes must have one.
This can make it easy to spot because the added data streams will need to have an name associated with them 
### Encryption Attribute 
Any Attribute could be encrypted but the $Data attribute (the content attribute) is the only attribute that gets encrypted.
When the attribute gets encrypted the header of the attribute remains unencrypted but the content of the attribute is encrypted.
A additional attribute is also created called the $Logged_utility_stream which contains the keys in order to decrypt the file.
A random key is created for each MFT entry with an encrypted attribute and this is called the file encryption key (FEK). If 
there are more than one data streams (Alternative Data Stream file) they all get encrypted with the same FEK. 

This FEK is stored in the $LOGGED_UTILITY_STREAM attribute. This attribute contains two lists of Data Decryption fields (DDF) and data Recovery fields (DRF).
A Data decryption field id created for each user who has access to the file and this field contains the following the users 
SID (Security ID), encryption information and the FEK that was used to encrypt the key encrypted with the users public key so that
only the users private key can decrypt the FEK and then decrypt the file. A data recovery field (DRF) is created for every method of data
recovery, and it contains the FEK encrypted with the data recovery public key which can then used when allow access to the FEK when
the data recovery private key is used to decrypt it and then can decrypt the file and recover it.

When a user is revoked from the ACL and can no longer access their key is removed from the data decryption field list, so they
can no longer decrypt the file. The Users private key is stored in the registry and is encrypted using a symmetric encryption 
(same key to encrypt and decrypt) and the key to encrypt the private key with is derived from the users' password. 
