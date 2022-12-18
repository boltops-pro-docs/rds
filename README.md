<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/rds/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# RDS CloudFormation Blueprint

[![Watch the video](https://img.boltops.com/boltopspro/video-preview/blueprints/rds/rds-mysql.png)](https://www.youtube.com/watch?v=mDU_Lm5xhig)

For videos with the other database engines like PostgreSQL, MariaDB, Microsoft SQL Server, Oracle go to the [Videos page](docs/videos.md).

![CodeBuild](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoidFBFcG9IaFVLZFNYa2V4YnZNWnVCTERhMklqQ3BINzY2aVhYL1k1T2UwMTNwYldLTWhpOHdRRjBubU40dGRUd21jdnhSbDQvaVdaLzFuLzhJSVBQNTlFPSIsIml2UGFyYW1ldGVyU3BlYyI6IjhiQk1iQkdwMjdRanRxSGoiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This blueprint provisions a RDS database.

* You can configure databases like MySQL, PostgreSQL, MariaDB, Microsoft SQL Server, Oracle, etc. For Aurora, refer to the [Aurora Blueprint](https://github.com/boltopspro-docs/aurora).  For Aurora Serverless, refer to the [Aurora Serverless Blueprint](https://github.com/boltopspro-docs/aurora-serverless).
* The default database is MySQL. Several [AWS::RDS::Instance properties](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-database-instance.html) are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Additionally, properties that require further customization are configurable with [Variables](https://lono.cloud/docs/configs/shared-variables/).  The blueprint is extremely flexible and configurable for your needs.
* Storage is encrypted by default.
* Managed DB Subnet Groups can be created by configuring `DBSubnets`. Or you can use an existing `DBSubnetGroupName`.
* Can create an optional managed Route53 record that points to the database endpoint.

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/rds values
3. Deploy blueprint

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "rds", git: "git@github.com:boltopspro/rds.git"
```

## Configure

Use the [lono seed](https://lono.cloud/reference/lono-seed/) command to generate a starter config params files.

    LONO_ENV=development lono seed rds
    LONO_ENV=production  lono seed rds

The files in `config/rds` folder will look something like this:

    configs/rds/
    ├── params
    │   ├── development.txt
    │   └── production.txt
    └── variables
        ├── development.rb
        └── production.rb

Configure the `configs/rds/params` and `configs/rds/variables` files.  The parameters required: `DbUser`, `DbPassword`, and `VpcId`.  Example:

configs/rds/params/development.txt:

    # Required parameters:
    DbUser=myuser
    DbPassword=mypassword
    VpcId=vpc-111
    # Optional parameters:
    # AllocatedStorage=100
    # CopyTagsToSnapshot=true
    # DBName=mydb # Must begin with a letter and contain only alphanumeric characters
    # DBInstanceClass=db.t3.micro
    # DBSnapshotIdentifier=...
    # DBSubnetGroupName=...
    # DBSubnets=subnet-111,subnet-222
    # DeletionProtection=false
    # Engine=mysql
    # Iops=1000
    # MonitoringInterval=1
    # MultiAZ=false
    # EnablePerformanceInsights=false
    # PubliclyAccessible=false
    # StorageEncrypted=true
    # StorageType=gp2 # More: standard, gp2, io1
    # ParameterGroupFamily=mysql5.7
    # VPCSecurityGroups=sg-111,sg-222
    # DnsName=rds.example.com.
    # HostedZoneId=/hostedzone/Z01963071SQH02EXAMPLE
    # HostedZoneName=example.com.
    # DnsType=CNAME
    # DnsTtl=60

configs/rds/variables/development.rb:

```ruby
@parameter_group_parameters = {} # parameter property for AWS::RDS::DBParameterGroup resource
@option_group_properties = {} # properties for AWS::RDS::OptionGroup resource
@database_properties = {} # properties AWS::RDS::DBInstance resource
```

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy.

    LONO_ENV=development lono cfn deploy rds --sure --no-wait
    LONO_ENV=production  lono cfn deploy rds --sure --no-wait

It takes about 15m to deploy the RDS DB instance. Times may vary.

If you are using One AWS Account, use these commands instead: [One Account](docs/one-account.md).

## Configure: More Details

### Compatible Parameters and Variables

The `Engine`, `EngineVersion`, `DBParameterGroupFamily`, `DBInstanceClass`, `LicenseModel` parameters must be a valid combination.

Engine | EngineVersion | DBParameterGroupFamily | DBInstanceClass | LicenseModel
--- | --- | --- | --- | ---
mysql | 8.0.16 | mysql8.0 | db.t3.micro | n/a
mysql | 5.7.26 | mysql5.7 | db.t3.micro | n/a
postgres | 11.5 | postgres11 | db.t3.micro | n/a
mariadb | 10.3.8 | mariadb10.3 | db.t3.micro | n/a
sqlserver-se | 14.00.3223.3.v1 | sqlserver-se-14.0 | db.t3.xlarge | license-included
oracle-se2 | 19.0.0.0.ru-2019-10.rur-2019-10.r1 | oracle-se2-19 | db.t3.xlarge | license-included

Each particular database engine is different and has its own configuration values. The quickest way to find what version are available to find out what parameters are compatible is to use the RDS console and pretend to create a DB and see the values. You can also refer to each databases's docs to find out what you can set. Here are some useful docs:

* [aws rds cli tips](docs/aws-rds-cli.md)
* [DBInstanceClass Support Matrix](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.DBInstanceClass.html#Concepts.DBInstanceClass.Support)
* [Oracle DBInstanceClass Support Matrix](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Oracle.html#Oracle.Concepts.InstanceClasses)

### Using a Custom VPC

To provision the database to a custom vpc, provide the `VpcId` parameter. You must also provide either `DBSubnetGroupName` or `DBSubnets`. Only provide only one, not both.

By Providing the `DBSubnets` parameter only the blueprint will create a managed [AWS::RDS::DBSubnetGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbsubnet-group.html) resource in the same custom VPC. Example:

    VpcId=vpc-111
    DBSubnets=subnet-111,subnet-222

### Private Data Subnet

It is recommended to run the database in a private data subnet. The [reference-architecture](https://github.com/boltopspro-docs/reference-architecture) [vpc](https://github.com/boltopspro-docs/vpc) blueprint is has a `PrivateDBSubnetGroup` db subnet group contains the private subnets.  A quick way to get the VPC and subnet values is from the VPC CloudFormation Outputs. Here's an example of the development VPC.

![](https://img.boltops.com/boltopspro/blueprints/vpc/dev-vpc-outputs.png)

If you are not using the reference architecture and you do not have a Db Subnet Group, specifying `DBSubnets` will create a [AWS::RDS::DBSubnetGroup
](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbsubnet-group.html) for you.

### Security Groups

To assign existing security groups to the RDS database use `VPCSecurityGroups`. Example:

    VPCSecurityGroups=sg-111,sg-222

If not set, then the blueprint will create and managed Security Group and assign to it to the RDS database.

### Route53 DNS Pretty Host Name

It is recommended to use a Private HostedZone to create a pretty endpoint.  Example:

    DnsName=rds.private.example.com.
    HostedZoneName=private.example.com.

### Advanced Properties Customizations

Several [AWS::RDS::Instance properties](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-database-instance.html) are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Properties that are not configurable with Parameters are configured with [Variables](https://lono.cloud/docs/configs/shared-variables/).  The `@database_properties` variable allows you to override any property.  Example:

```ruby
@database_properties = {
  port: 3306,
  character_set_name: "utf8_unicode_ci",
}
```

The blueprint is written so that Variables take higher precedence than Parameters.

### Creating Replicas

You can use this same blueprint to create a replica with another RDS database.  Make sure to not configure the `DBSubnetGroupName` or `DBSubnets` parameters. When creating a replica, RDS does not allow setting the DB Subnet group as it inherits the setting from the master.  You will get an error that looks like this:

> DBSubnetGroupName rds should not be specified for read replicas that are created in the same region as the master

### DB Parameter Group and DB Option Group

Both DB Parameter Group and DB Option Group can be configured with variables:

```ruby
@db_parameters: {
  sql_mode: "IGNORE_SPACE"
}
@db_options = {
  engine_name: "mysql",
  major_engine_version: "5.7",
  option_configurations: [
    {
      option_name: "MEMCACHED",
      option_settings: [
        {
          name: "CHUNK_SIZE",
          value: "32"
        },
        {
          name: "BINDING_PROTOCOL",
          value: "ascii"
        }
      ],
      port: "1234",
      vpc_security_group_memberships: [
        "sg-111"
      ]
    }
  ],
  option_group_description: "A test option group"
}
```

Note: When configuring `@db_options` and later deleting the RDS stack, CloudFormation fails to delete the DB Option Group because the final DB snapshot references the DB Option Group.  To delete the stack successfully, delete all the snapshots associated with the DB, and then you'll be able to delete the stack cleanly. You can also skip the deletion of the DB Option Group resource and manually clean up later if you choose to do so.

## Considerations

Using CloudFormation to provision RDS databases has some pros and cons.

* The main pro is that the infrastructure is codified and reproducible.
* The main con is that CloudFormation can **replace** your database entirely, and you'll lose your data!

Depending on the [AWS::RDS::Instance property](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-database-instance.html) property that is changed, CloudFormation **replaces** the database entirely. You have to look for "Update requires: Replacement". This is dangerously easy to forget.

An excellent way to provide a guard rail against accidental replacement is to set the `DBInstanceIdentifier` parameter. If you change any property that requires replacement, then the CloudFormation stack update will immediately fail, because RDS won't be able to create another database with the same `DBInstanceIdentifier`.

Another technique you can use to prevent an accidental replacement is only to use CloudFormation for the initial provisioning of the database. Afterward, you'll modify the RDS DB with the API or Console only. Yet another way to provide a guard rail is to enable `DeletionProtection=true`. This prevents the current database from being deleted. However, it deletion happens as part of the CloudFormation cleanup step, so it takes longer for the rollback to finish.
