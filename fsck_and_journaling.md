# FSCK and Journaling

fsck is short for "file system consistency check". Journaling js also known as write-ahead logging.

**EXAMPLE : ** We want to append a 4k bytes content to existing file, so we have to update the inode map, data block map and one data block, which require 3 writes. If after 1 or 2 write, the system crashed, then we may face an file system inconsistency or space leak.

##Solutions

1. 


