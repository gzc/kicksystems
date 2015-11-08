# FSCK and Journaling

fsck is short for "file system consistency check". Journaling js also known as write-ahead logging.

**EXAMPLE : ** We want to append a 4k bytes content to existing file, so we have to update the inode map, data block map and one data block, which require 3 writes. If after 1 or 2 write, the system crashed, then we may face an file system inconsistency or space leak.

##Solutions

###1. FSCK

fsck is a UNIX tool for finding such inconsistencies
and repairing them. However, fsck can not deal with all the cases. If we only update inode structure but haven't written data to block, then the file system looks consistent but the inode points to garbage data. The only real goal of fsck is to make sure the file system metadata is internally consistent.

###2. Journaling (or Write-Ahead Logging)
TxB is the start block of transaction, and TxE is the end.

1. Data write: Write data to final location; wait for completion
(the wait is optional; see below for details).
2. Journal metadata write: Write the begin block and metadata to the
log; wait for writes to complete.
3. Journal commit: Write the transaction commit block (containing
TxE) to the log; wait for the write to complete; the transaction (including
data) is now committed.
4. Checkpoint metadata: Write the contents of the metadata update
to their final locations within the file system.
5. Free: Later, mark the transaction free in journal superblock.

In Linux **ex4**, 1 and 2 are merged by checksum which makes writing faster.

When crashed, we can apply **redo logging.** 

**Batching Log Updates** is a trick to handle excessive write. For example, if a file inode and its directory inode are in the same block, we can't update them individually, we'd better batch log updates and write once.

###3. Other Approaches


