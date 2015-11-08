# FSCK and Journaling

fsck is short for "file system consistency check". Journaling js also known as write-ahead logging.

**EXAMPLE : ** We want to append a 4k bytes content to existing file, so we have to update the inode map, data block map and one data block, which require 3 writes. If after 1 or 2 write, the system crashed, then we may face an file system inconsistency or space leak.

##Solutions

###1. FSCK

fsck is a UNIX tool for finding such inconsistencies
and repairing them. However, fsck can not deal with all the cases. If we only update inode structure but haven't written data to block, then the file system looks consistent but the inode points to garbage data. The only real goal of fsck is to make sure the file system metadata is internally consistent.

###2. Journaling (or Write-Ahead Logging)
TxB is the start block of transaction, and TxE is the end.

1. Journal write: Write the contents of the transaction (including TxB,
metadata, and data) to the log; wait for these writes to complete.
2. Journal commit: Write the transaction commit block (containing
TxE) to the log; wait for write to complete; transaction is said to be
committed.
3. Checkpoint: Write the contents of the update (metadata and data)
to their final on-disk locations.

In Linux **ex4**, 1 and 2 are merged by checksum which makes writing faster.

When crashed, we can apply **redo logging.** 

**Batching Log Updates** is a trick to handle excessive write.


