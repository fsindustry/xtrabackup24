==========================
Frequently Asked Questions
==========================

Do I need an InnoDB Hot Backup license to use Percona XtraBackup?
=================================================================

No. Although ``innobackupex`` is derived from the same GPL and open-source
wrapper script that InnoDB Hot Backup uses, it does not execute ``ibbackup``,
and the ``xtrabackup`` binary does not execute or link to ``ibbackup``. You
can use |Percona XtraBackup| without any license; it is completely separate
from InnoDB Hot Backup.

What's the difference between :program:`innobackupex` and
=========================================================
:program:`innobackup`?
======================

Because :program:`innobackupex` is a patched version of *Oracle* ’s
:program:`innobackup` script (now renamed to :program:`mysqlbackup`), it is
quite similar in some ways, and familiarity with innobackup might be helpful.

Aside from the options for specific features of |innobackupex|, the main
differences are:

  * printing to ``STDERR`` instead of ``STDOUT`` (which enables the
    :option:`innobackupex --stream` option),

  * the configuration file - :term:`my.cnf` - is detected automatically (or
    set with :option:`innobackupex --defaults-file`) instead of the mandotory
    first argument,

  * and defaults to |xtrabackup| as binary to use in the :option:`innobackupex --ibbackup`.

See :doc:`innobackupex/innobackupex_option_reference` for more details.

Are you aware of any web-based backup management tools (commercial or not)
==========================================================================
built around |Percona XtraBackup|?
==================================

`Zmanda Recovery Manager <http://www.zmanda.com/zrm-mysql-enterprise.html>`_ is
a commercial tool that uses |Percona XtraBackup| for Non-Blocking Backups:

 *"ZRM provides support for non-blocking backups of MySQL using |Percona
 XtraBackup|. ZRM with |Percona XtraBackup| provides resource utilization
 management by providing throttling based on the number of IO operations per
 second. |Percona XtraBackup| based backups also allow for table level recovery
 even though the backup was done at the database level (needs the recovery
 database server to be |Percona Server| with XtraDB)."*

|xtrabackup| binary fails with a floating point exception
=========================================================

In most of the cases this is due to not having install the required libraries
(and version) by |xtrabackup|. Installing the *GCC* suite with the supporting
libraries and recompiling |xtrabackup| will solve the issue. See
:doc:`installation/compiling_xtrabackup` for instructions on the procedure.

How xtrabackup handles the ibdata/ib_log files on restore if they aren't in
===========================================================================
mysql datadir?
==============

In case the :file:`ibdata` and :file:`ib_log` files are located in different
directories outside of the datadir, you will have to put them in their proper
place after the logs have been applied.

Backup fails with Error 24: 'Too many open files'
=================================================

This usually happens when database being backed up contains large amount of
files and |Percona XtraBackup| can't open all of them to create a successful
backup. In order to avoid this error the operating system should be configured
appropriately so that |Percona XtraBackup| can open all its files. On Linux,
this can be done with the ``ulimit`` command for specific backup session or by
editing the :file:`/etc/security/limits.conf` to change it globally (**NOTE**:
the maximum possible value that can be set up is ``1048576`` which is a
hard-coded constant in the Linux kernel).

How to deal with skipping of redo logs for DDL operations?
==========================================================

To prevent creating corrupted backups when running DDL operations,
Percona XtraBackup aborts if it detects that redo logging is disabled.
In this case, the following error is printed::

 [FATAL] InnoDB: An optimized (without redo logging) DDL operation has been performed. All modified pages may not have been flushed to the disk yet.
 Percona XtraBackup will not be able to take a consistent backup. Retry the backup operation.

.. note:: Redo logging is disabled during a `sorted index build
   <https://dev.mysql.com/doc/refman/5.7/en/sorted-index-builds.html>`_

To avoid this error,
Percona XtraBackup can use metadata locks on tables while they are copied:

* To block all DDL operations, use the :option:`xtrabackup --lock-ddl` option
  that issues ``LOCK TABLES FOR BACKUP``.

* If ``LOCK TABLES FOR BACKUP`` is not supported, you can block DDL for each
  table before XtraBackup starts to copy it and until the backup is completed
  using the :option:`xtrabackup --lock-ddl-per-table` option.

.. seealso::

    :ref:`lock_redesign`
