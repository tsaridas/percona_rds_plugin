Nagios plugin for Amazon RDS monitoring.

This program is part of Percona Monitoring Plugins
License: GPL License (see COPYING)

Author Roman Vynar
Copyright 2014 Percona LLC and/or its affiliates


pmp-check-aws-rds.py - Check Amazon RDS metrics.



```
  Usage: pmp-check-aws-rds.py [options]
  Options:
    -h, --help            show this help message and exit
    -l, --list            list of all DB instances
    -i IDENT, --ident=IDENT
                          DB instance identifier
    -p, --print           print status and other details for a given DB instance
    -m METRIC, --metric=METRIC
                          metric to check: [status, load, storage, memory]
    -w WARN, --warn=WARN  warning threshold
    -c CRIT, --crit=CRIT  critical threshold
    -u UNIT, --unit=UNIT  unit of thresholds for "storage" and "memory" metrics:
                          [percent, GB]. Default: percent
```
REQUIREMENTS 
This plugin is written on Python and utilizes the module C<boto> (Python interface
to Amazon Web Services) to get various RDS metrics from CloudWatch and compare
them against the thresholds.
* Install the package: C<yum install python-boto> or C<apt-get install python-boto>
* Create a config /etc/boto.cfg or ~nagios/.boto with your AWS API credentials.
  See http://code.google.com/p/boto/wiki/BotoConfig
This plugin that is supposed to be run by Nagios, i.e. under ``nagios`` user,
should have permissions to read the config /etc/boto.cfg or ~nagios/.boto.
Example:
```
  [root@centos6 ~]# cat /etc/boto.cfg
  [Credentials]
  aws_access_key_id = THISISATESTKEY
  aws_secret_access_key = thisisatestawssecretaccesskey
```
If you do not use this config with other tools such as our Cacti script,
you can secure this file the following way:
```
  [root@centos6 ~]# chown nagios /etc/boto.cfg
  [root@centos6 ~]# chmod 600 /etc/boto.cfg
```
DESCRIPTION

The plugin provides 4 checks and some options to list and print RDS details:
* RDS Status
* RDS Load Average
* RDS Free Storage
* RDS Free Memory
To get the list of all RDS instances under AWS account:
```
  # ./aws-rds-nagios-check.py -l
```
To get the detailed status of RDS instance identified as C<blackbox>: 
```
  # ./aws-rds-nagios-check.py -i blackbox -p
```
Nagios check for the overall status. Useful if you want to set the rest
of the checks dependent from this one:
```
  # ./aws-rds-nagios-check.py -i blackbox -m status
  OK mysql 5.1.63. Status: available
```
Nagios check for CPU utilization, specify thresholds as percentage of
1-min., 5-min., 15-min. average accordingly: 
```
  # ./aws-rds-nagios-check.py -i blackbox -m load -w 90,85,80 -c 98,95,90
  OK Load average: 18.36%, 18.51%, 15.95% | load1=18.36;90.0;98.0;0;100 load5=18.51;85.0;95.0;0;100 load15=15.95;80.0;90.0;0;100
```
Nagios check for the free memory, specify thresholds as percentage:
```
  # ./aws-rds-nagios-check.py -i blackbox -m memory -w 5 -c 2
  OK Free memory: 5.90 GB (9%) of 68 GB | free_memory=8.68;5.0;2.0;0;100
  # ./aws-rds-nagios-check.py -i blackbox -m memory -u GB -w 4 -c 2
  OK Free memory: 5.90 GB (9%) of 68 GB | free_memory=5.9;4.0;2.0;0;68
```
Nagios check for the free storage space, specify thresholds as percentage or GB: 
```
  # ./aws-rds-nagios-check.py -i blackbox -m storage -w 10 -c 5
  OK Free storage: 162.55 GB (33%) of 500.0 GB | free_storage=32.51;10.0;5.0;0;100
  # ./aws-rds-nagios-check.py -i blackbox -m storage -u GB -w 10 -c 5
  OK Free storage: 162.55 GB (33%) of 500.0 GB | free_storage=162.55;10.0;5.0;0;500.0
```
CONFIGURATION

Here is the excerpt of potential Nagios config:
```
  define servicedependency{
        hostgroup_name                  mysql-servers
        service_description             RDS Status
        dependent_service_description   RDS Load Average, RDS Free Storage, RDS Free Memory 
        execution_failure_criteria      w,c,u,p
        notification_failure_criteria   w,c,u,p
        }
  
  define service{
        use                             active-service
        hostgroup_name                  mysql-servers
        service_description             RDS Status
        check_command                   check_rds!status!0!0
        }
       
  define service{
        use                             active-service
        hostgroup_name                  mysql-servers
        service_description             RDS Load Average
        check_command                   check_rds!load!90,85,80!98,95,90
        }
  
  define service{
        use                             active-service
        hostgroup_name                  mysql-servers
        service_description             RDS Free Storage
        check_command                   check_rds!storage!10!5
        }
  
  define service{
        use                             active-service
        hostgroup_name                  mysql-servers
        service_description             RDS Free Memory
        check_command                   check_rds!memory!5!2
        }
  
  define command{
        command_name    check_rds
        command_line    $USER1$/pmp-check-aws-rds.py -i $HOSTALIAS$ -m $ARG1$ -w $ARG2$ -c $ARG3$
        }
```
COPYRIGHT, LICENSE, AND WARRANTY
This program is copyright 2014 Percona LLC and/or its affiliates.
Feedback and improvements are welcome.
THIS PROGRAM IS PROVIDED "AS IS" AND WITHOUT ANY EXPRESS OR IMPLIED
WARRANTIES, INCLUDING, WITHOUT LIMITATION, THE IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.
This program is free software; you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, version 2.  You should have received a copy of the GNU General
Public License along with this program; if not, write to the Free Software
Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA.

Percona Monitoring Plugins pmp-check-aws-rds.py 1.1.4
