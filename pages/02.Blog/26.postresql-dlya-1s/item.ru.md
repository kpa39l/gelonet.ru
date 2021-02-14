---
title: 'PostreSQL для 1С'
date: '18:50 28-11-2012'
publish_date: '18:50 28-11-2012'
taxonomy:
    category:
        - Blog
    tag:
        - Linux
---

Подбираю информацию по развертыванию PostgreSQL на Linux. Начал с подбора ФС. Вот что пишут в http://momjian.us/main/writings/pgsql/hw_performance/:
 Some operating systems support multiple disk file systems. In such cases, it can be difficult to know which file system performs best. POSTGRESQL usually performs best on traditional Unix file systems like the BSD UFS/FFS filesystems, which many operating systems support. The default 8K block size of UFS is the same as POSTGRESQL's page size. You can run on journal and log-based file systems, but these cause extra overhead during fsync's of the write-ahead log. Older SvR3-based file systems become too fragmented to yield good performance.

File system choice is particularly difficult on Linux because there are so many file system choices, and none of them are optimal: ext2 is not entirely crash-safe, ext3, XFS, and JFS are journal-based, and Reiser is optimized for small files and does journalling. The journalling file systems can be significantly slower than ext2 but when crash recovery is required, ext2 isn't an option. If ext2 must be used, mount it with sync enabled. Some people recommend an ext3 filesystem mounted with data=writeback.

Пока для себя вынес, что если использовать дисковый кэш записи, то обязательно с батарейкой, на случай перебоев. А его лучше использовать,поскольку при транзакции система ждет отклика об окончании записи данных в лог.

Про ext4 ничего не написано. Придется дальше искать опыт использования этой ФС на боевом сервере.