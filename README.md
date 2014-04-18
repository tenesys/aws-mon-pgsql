# aws-mon-pgsql

Bash script that reports custom metric data about PostgreSQL(RDS) performance to Amazon CloudWatch

This script is intended for use with Amazon EC2 instances running Linux operating systems for monitoring Amazon RDS for PostgreSQL instances. 

The scripts have been tested on the following Amazon Machine Images (AMIs) for both 32-bit and 64-bit versions:

* Amazon Linux 2014.3
* Red Hat Enterprise Linux 6.4
* Ubuntu Server 13.10

The scripts have been tested on the following RDS engines:

* PostgreSQL 9.3.3

The script is written in simple Bash, so you can use it very easily. 


## Metrics of This Script

This script can report the following metrics:

* status
* cache hit
* session
* transaction
* lock
* tupple read
* tupple write
* block read
* buffer write


## Prerequisites

This script requires [Amazon CloudWatch Command Line Tool](http://aws.amazon.com/developertools/2534).

### Amazon Linux

Instances that you launch using the Amazon Linux AMI already include the CLI tools.

All you have to do is to install git.

```
yum install git
```


### RHEL, Ubuntu

#### RHEL

Install git.

```
yum install git
```

#### Ubuntu

Install git, unzip and jre.

```
apt-get install git
apt-get install unzip
apt-get install default-jre
```

In order to set correct JAVA_HOME, you have to edit aws-mon.sh as follows:

```
export JAVA_HOME=/usr/lib/jvm/default-java/
```


#### RHEL, Ubuntu

You can install [Amazon CloudWatch Command Line Tool](http://aws.amazon.com/developertools/2534) with following steps:

```
wget http://ec2-downloads.s3.amazonaws.com/CloudWatch-2010-08-01.zip
unzip CloudWatch-2010-08-01.zip
mkdir -p /opt/aws/apitools/
mv CloudWatch-1.0.20.0 /opt/aws/apitools/mon-1.0.20.0
cd /opt/aws/apitools
ln -s mon-1.0.20.0 mon
mkdir /opt/aws/bin
cd /opt/aws/bin
ln -s /opt/aws/apitools/mon-1.0.20.0/bin/mon-put-data mon-put-data
```

## Getting Started

Download "aws-mon-pgsql.sh" or clone repository using following steps: 

```
git clone https://github.com/moomindani/aws-mon-pgsql.git
cd aws-mon-pgsql
./aws-mon-pgsql.sh --help
```

## Using the Script

This script collects status, cache hit, session, transaction, lock, tupple read, tupple write, block read, buffer write data on the target RDS. It then makes a remote call to Amazon CloudWatch to report the collected data as custom metrics.

### Option

Name                       | Description
-------------------------- | -------------------------------------------------
--help                     | Displays usage information.
--version                  | Displays the version number.
--verify                   | Checks configuration and prepares a remote call.
--verbose                  | Displays details of what the script is doing.
--debug                    | Displays information for debugging.
--from-cron                | Use this option when calling the script from cron.
--aws-credential-file PATH | Provides the location of the file containing AWS credentials. This parameter cannot be used with the --aws-access-key-id and --aws-secret-key parameters.
--aws-access-key-id VALUE  | Specifies the AWS access key ID to use to identify the caller. Must be used together with the --aws-secret-key option. Do not use this option with the --aws-credential-file parameter.
--aws-secret-key VALUE     | Specifies the AWS secret access key to use to sign the request to CloudWatch. Must be used together with the --aws-access-key-id option. Do not use this option with --aws-credential-file parameter.
--id	                     | Specifies database instance identifier.
-h	                       | Specifies database server host.
-p	                       | Specifies database server port.
-U	                       | Specifies database user.
-d	                       | Specifies database name.
--status-check-failed      | Reports whether the database instance has passed the status check.
--status-check-timeout     | Specifies status check timeout. (default: 10s)
--cache-hit	               | Reports cache hit ratio.
--session-active	         | Reports the number of sessions whose status is active.
--session-idle	           | Reports the number of sessions whose status is idle.
--session-wait	           | Reports the number of sessions whose status is wait.
--txn-commit	             | Reports the number of transactions committed. (Run at least twice with same options)
--txn-rollback	           | Reports the number of transactions rollbacked. (Run at least twice with same options)
--locks-acquired	         | Reports the number of locks acquired.
--locks-wait	             | Reports the number of locks wait.
--tup-inserted             | Reports the number of tupples inserted. (Run at least twice with same options)
--tup-updated	             | Reports the number of tupples updated. (Run at least twice with same options)
--tup-deleted	             | Reports the number of tupples deleted. (Run at least twice with same options)
--tup-returned	           | Reports the number of tupples returned. (Run at least twice with same options)
--tup-fetched	             | Reports the number of tupples fetched. (Run at least twice with same options)
--blks-read	               | Reports the number of blocks not included in shared memory and read from disk. (Needs to run at least twice with same options)
--blks-hit	               | Reports the number of blocks included in shared memory. (Run at least twice with same options)
--buffers-checkpoint	     | Reports the number of buffers written for checkpoint. (Run at least twice with same options)
--buffers-clean	           | Reports the number of buffers written for cleaning. (Run at least twice with same options)
--buffers-backend	         | Reports the number of buffers written for backend. (Run at least twice with same options)
--all-items                | Reports all items.

### Examples

The following examples assume that you have already updated the awscreds.conf file with valid AWS credentials. If you are not using the awscreds.conf file, provide credentials using the --aws-access-key-id and --aws-secret-key arguments.

#### To perform a simple test run without posting data to CloudWatch

* Run the following command:

```
./aws-mon-pgsql.sh --id postgres -h postgres.xxx.ap-northeast-1.rds.amazonaws.com -p 5432 -U postgres -d postgres --status-check-failed --verify --verbose
```

#### To collect the status and send them to CloudWatch

* Run the following command:

```
./aws-mon-pgsql.sh --id postgres -h postgres.xxx.ap-northeast-1.rds.amazonaws.com -p 5432 -U postgres -d postgres --status-check-failed  --verbose
```

#### To collect all available session metrics and send them to CloudWatch

* Run the following command:

```
./aws-mon-pgsql.sh --id postgres -h postgres.xxx.ap-northeast-1.rds.amazonaws.com -p 5432 -U postgres -d postgres --session-active --session-idle --session-wait --verbose
```

#### To collect the cache hit ratio and send them to CloudWatch

* Run the following command:

```
./aws-mon-pgsql.sh --id postgres -h postgres.xxx.ap-northeast-1.rds.amazonaws.com -p 5432 -U postgres -d postgres --cache-hit --verbose
```

#### To collect all available tupple metrics and send them to CloudWatch

* Run the following command:

```
./aws-mon-pgsql.sh --id postgres -h postgres.xxx.ap-northeast-1.rds.amazonaws.com -p 5432 -U postgres -d postgres --tup-inserted --tup-updated --tup-deleted --tup-returned --tup-fetched --verbose
(wait a few seconds...)
./aws-mon-pgsql.sh --id postgres -h postgres.xxx.ap-northeast-1.rds.amazonaws.com -p 5432 -U postgres -d postgres --tup-inserted --tup-updated --tup-deleted --tup-returned --tup-fetched --verbose
```

#### To set a cron schedule for metrics reported to CloudWatch

1. Start editing the crontab using the following command:

```
crontab -e
```

2. Add the following command to report all items to CloudWatch every five minutes:

```
*/5 * * * *  ~/aws-mon-pgsql/aws-mon-pgsql.sh --id postgres -h postgres.xxx.ap-northeast-1.rds.amazonaws.com -p 5432 -U postgres -d postgres --aws-credential-file ~/awscreds --all-items --from-cron
```
