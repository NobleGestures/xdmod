---
title: Frequently Asked Questions
---

### What are the software requirements for Open XDMoD?

See [Software Requirements](software-requirements.html) for details.

### What are the hardware requirements for Open XDMoD?

The hardware requirements for Open XDMoD depend on how many concurrent
users you have and how much data you have.  You'll need roughly 300MB of
disk space per 1 Million jobs on your MySQL server.

### Will Open XDMoD run on RHEL 5?

It is possible, but you must have PHP 5.3 installed.

### Can I use Open XDMoD with MySQL 5.0?

Open XDMoD should work with MySQL 5.0, but it hasn't been tested
extensively, so we recommend MySQL 5.1 or 5.5.

### How do I enable LDAP Authentication?

Open XDMoD has support for [federated authentication](simpleSAMLphp.html)
using [simpleSAMLphp][simplesaml].  Basic support for [LDAP Authentication](simpleSAMLphp-ldap.html) is also provided.

[simplesaml]: https://simplesamlphp.org/

### Why does the ingestion process take a long time?

If you are experiencing long ingestion times make sure that you have
tuned MySQL properly.  See
[Optimizing the MySQL Server][optimizing-mysql] for more details.

Here is an example of some server parameters that you can change.  Be
sure to understand any changes you make to your MySQL server
configuration.

    [mysqld]
    key_buffer_size         = 1G
    sort_buffer_size        = 8M
    read_buffer_size        = 4M
    join_buffer_size        = 16M
    innodb_buffer_pool_size = 64M
    max_allowed_packet      = 16M
    tmp_table_size          = 1G
    max_heap_table_size     = 128M
    thread_stack            = 256K
    thread_cache_size       = 8
    query_cache_limit       = 16M
    query_cache_size        = 1G

[optimizing-mysql]: https://dev.mysql.com/doc/refman/5.5/en/optimizing-the-server.html

### How do I install Open XDMoD in a non-root URL?

Non-root URLs are not supported at this time.

### How do I change the charts on the summary page?

See [HOWTO Change Summary Page Charts](howto-summary-charts.html) for
details.

### How do I add more user names after the initial import?

Just repeat the process found [here](user-names.html) with the new user
names.

### How do I delete all my job data from Open XDMoD?

If you think the job data in your Open XDMoD database is corrupted, you
may want to delete your job data and start over.  To do so, use this
command:

     $ xdmod-admin --truncate --jobs

Running this command will truncate all the tables containing job data
and you can then re-shred and re-ingest your resource manager data.

### Why do I see the error message "It is not safe to rely on the system's timezone settings..."?

You need to set your timezone in your `php.ini` file.  Add the
following, but substitute your timezone:

    date.timezone = America/New_York

The PHP website contains the full list of supported [timezones][].

[timezones]: http://php.net/manual/en/timezones.php

### Why do I see "Unknown resource attribute" in my xdmod-shredder output?

This indicates that you are using a resource attribute that Open XDMoD
does not recognize. This isn't a problem and can be safely ignored.

### Why do I see the error message "ERROR 1419 (HY000) at line 290: You do not have the SUPER privilege and binary logging is enabled..."?

You have [binary logging][mysql-binary-log] enabled, but the user you
specified to create the Open XDMoD databases doesn't have the `SUPER`
privilege.  You should either disable binary logging (assuming you don't
need it) or [grant][mysql-grant] the `SUPER` privilege to the user that
will create the databases.  You may also use the less safe
[log_bin_trust_function_creators][] variable.

[mysql-binary-log]:                https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html
[mysql-grant]:                     https://dev.mysql.com/doc/refman/5.5/en/grant.html
[log_bin_trust_function_creators]: https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#option_mysqld_log-bin-trust-function-creators

### Why do I see the error message "Failed to correct job times" during the shredding process?

Resource manager account log records may be missing some job data under certain
circumstances.  It is expected that a job record will have a start time, end
time and wall time.  If one of these is missing it will be calculated using the
other two.  If two or more are missing the error message in question will be
displayed along with the values that were present in the job record.  In
addition to this error message a file will be produced at the end of the
shredding process containing the job record that produced the error along with
additional details.  Look for the log message "Job errors written to ...".

One example of a situation that will produce this error is that some resource
managers record a 0 start time in their accounting logs when a job is canceled
before the job started.  It is also possible that other errors occurred to
produce these values so they are not ignored and this error message is produced.
As of Open XDMoD 7.0 any job with a start or end time equal to 0 will not be
ingested into the data warehouse and will not contribute to the job count or
any other metrics.  Previous versions of Open XDMoD did include these jobs
which resulted in inaccurate results.

### Why do I see the error message "User "Public User" not found" instead of the portal?

This indicates that the data needed by the new ACL system introduced in 7.0.0 is
not present.  This is typically caused by failing to migrate the Open XDMoD
database from 6.6.0 to 7.0.0.  The `xdmod-upgrade` script must be executed
after upgrading the Open XDMoD RPM or source installation.
