.. _backup:

备份
======

备份Orthanc的方法取决于数据库后端的使用。无论如何，你当然得备份你的
:ref:`配置文件<configuration>`.

SQLite
------

默认情况下，Orthanc使用SQLite来存储数据库。在这种情况下,
所有的DICOM文件和SQLite索引都是直接存储在文件系统中。备份过程如下所示:

1. 停止Orthanc.
2. 复制下面三部分文件:

   * 你的配置文件.
   * DICOM文件 (默认情况下，在配置文件同一目录的 ``OrthancStorage`` 文件夹下).
   * SQLite 索引 (默认情况下，在配置文件同一目录的 ``OrthancStorage/index*`` 一系列文件).

3. 重启Orthanc.

停止Orthanc是强制性的，因为Orthanc核心在任何时间都假设它是访问SQLite数据库的惟一过程。

Karsten Hilbert 给我提供了Orthanc官方Debian包的自动备份过程的 `备份脚本示例
<https://github.com/jodogne/OrthancContributed/blob/master/Scripts/Backup/2014-01-31-KarstenHilbert.sh>`__
。注意，在这个脚本中，调用SQLite命令行工具强制用`WAL replay
<http://www.sqlite.org/wal.html>`__.


PostgreSQL
----------

默认的SQLite引擎非常适合DICOM路由或图像缓冲任务，但不适用于企业场景。
在这种情况下，强烈建议您使用 `PostgreSQL后端
<http://www.orthanc-server.com/static.php?page=postgresql>`__.

如果使用PostgreSQL，则可以进行热备份 (例如， Orthanc正在运行), 并且您可以从PostgreSQL备份的所有灵活性中获益。
这些规程超出了本手册的范围，请参考 `官方备份和恢复手册
<https://www.postgresql.org/docs/devel/static/backup.html>`__.
