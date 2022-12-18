<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/rds/blob/master/docs/aws-rds-cli.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# RDS DB Engine Versions

RDS supports several different types of databases and versions. This doc provides a tutorial on how to query for them with the [aws rds describe-db-engine-versions](https://docs.aws.amazon.com/cli/latest/reference/rds/describe-db-engine-versions.html) cli.

## Useful Commands

We'll save a list of all db engine versions to a file. This will speed up future searches by avoiding the API call.

    aws rds describe-db-engine-versions > db-engine-versions.json

To get a list of all db engines support:

    $ cat db-engine-versions.json | jq -r '.DBEngineVersions[].Engine' | sort | uniq
    aurora
    aurora-mysql
    aurora-postgresql
    docdb
    mariadb
    mysql
    neptune
    oracle-ee
    oracle-se
    oracle-se1
    oracle-se2
    postgres
    sqlserver-ee
    sqlserver-ex
    sqlserver-se
    sqlserver-web

Take a specific db engine and get all the versions it supports. As an example, here's mysql:

    $ cat db-engine-versions.json | jq -r '.DBEngineVersions[] | select(.Engine == "aurora-mysql") | .EngineVersion' | sort | uniq
    5.7.12
    5.7.mysql_aurora.2.03.2
    5.7.mysql_aurora.2.07.0
    5.7.mysql_aurora.2.07.1

Finding all the DBParameterGroupFamily values is also useful:

    $ cat db-engine-versions.json | jq -r '.DBEngineVersions[].DBParameterGroupFamily' | sort | uniq | grep aurora
    aurora5.6
    aurora-mysql5.7
    aurora-postgresql10
    aurora-postgresql11
    aurora-postgresql9.6

To get a list db parameters:

    aws rds describe-db-parameters --db-parameter-group-name default.aurora-mysql5.7 | jq -r '.Parameters[].ParameterName'
    aws rds describe-db-parameters --db-parameter-group-name default.aurora-postgresql11 | jq -r '.Parameters[].ParameterName'
