
# terraform-aws-rds

Terraform module to provision AWS [`RDS`](https://aws.amazon.com/rds/) instances



## Introduction

The module will create:

* DB instance (MySQL, Postgres, SQL Server, Oracle)
* DB Option Group (will create a new one or you may use an existing)
* DB Parameter Group
* DB Subnet Group
* DB Security Group
* DNS Record in Route53 for the DB endpoint

## Usage


**IMPORTANT:** The `master` branch is used in `source` just as an example. In your code, do not pin to `master` because there may be breaking changes between releases.
Instead pin to the release tag (e.g. `?ref=tags/x.y.z`) of one of our [latest releases](https://github.com/ceibo-it/terraform-aws-rds/releases).


```hcl
module "rds_instance" {
    source                      = "git::https://github.com/ceibo-it/terraform-aws-rds.git?ref=master"
    namespace                   = "eg"
    stage                       = "prod"
    name                        = "app"
    dns_zone_id                 = "Z89FN1IW975KPE"
    host_name                   = "db"
    security_group_ids          = ["sg-xxxxxxxx"]
    ca_cert_identifier          = "rds-ca-2019"
    allowed_cidr_blocks         = ["XXX.XXX.XXX.XXX/32"]
    database_name               = "wordpress"
    database_user               = "admin"
    database_password           = "xxxxxxxxxxxx"
    database_port               = 3306
    multi_az                    = true
    storage_type                = "gp2"
    allocated_storage           = 100
    storage_encrypted           = true
    engine                      = "mysql"
    engine_version              = "5.7.17"
    major_engine_version        = "5.7"
    instance_class              = "db.t2.medium"
    db_parameter_group          = "mysql5.7"
    option_group_name           = "mysql-options"
    publicly_accessible         = false
    subnet_ids                  = ["sb-xxxxxxxxx", "sb-xxxxxxxxx"]
    vpc_id                      = "vpc-xxxxxxxx"
    snapshot_identifier         = "rds:production-2015-06-26-06-05"
    auto_minor_version_upgrade  = true
    allow_major_version_upgrade = false
    apply_immediately           = false
    maintenance_window          = "Mon:03:00-Mon:04:00"
    skip_final_snapshot         = false
    copy_tags_to_snapshot       = true
    backup_retention_period     = 7
    backup_window               = "22:00-03:00"

    db_parameter = [
      { name  = "myisam_sort_buffer_size"   value = "1048576" },
      { name  = "sort_buffer_size"          value = "2097152" }
    ]

    db_options = [
      { option_name = "MARIADB_AUDIT_PLUGIN"
          option_settings = [
            { name = "SERVER_AUDIT_EVENTS"           value = "CONNECT" },
            { name = "SERVER_AUDIT_FILE_ROTATIONS"   value = "37" }
          ]
      }
    ]
}
```



## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| allocated_storage | The allocated storage in GBs | number | - | yes |
| allow_major_version_upgrade | Allow major version upgrade | bool | `false` | no |
| allowed_cidr_blocks | The whitelisted CIDRs which to allow `ingress` traffic to the DB instance | list(string) | `<list>` | no |
| apply_immediately | Specifies whether any database modifications are applied immediately, or during the next maintenance window | bool | `false` | no |
| associate_security_group_ids | The IDs of the existing security groups to associate with the DB instance | list(string) | `<list>` | no |
| attributes | Additional attributes (e.g. `1`) | list(string) | `<list>` | no |
| availability_zone | The AZ for the RDS instance | string | `null` | no |
| auto_minor_version_upgrade | Allow automated minor version upgrade (e.g. from Postgres 9.5.3 to Postgres 9.5.4) | bool | `true` | no |
| backup_retention_period | Backup retention period in days. Must be > 0 to enable backups | number | `0` | no |
| backup_window | When AWS can perform DB snapshots, can't overlap with maintenance window | string | `22:00-03:00` | no |
| ca_cert_identifier | The identifier of the CA certificate for the DB instance | string | `rds-ca-2019` | no |
| copy_tags_to_snapshot | Copy tags from DB to a snapshot | bool | `true` | no |
| database_name | The name of the database to create when the DB instance is created | string | - | yes |
| database_password | (Required unless a snapshot_identifier or replicate_source_db is provided) Password for the master DB user | string | `` | no |
| database_port | Database port (_e.g._ `3306` for `MySQL`). Used in the DB Security Group to allow access to the DB instance from the provided `security_group_ids` | number | - | yes |
| database_user | (Required unless a `snapshot_identifier` or `replicate_source_db` is provided) Username for the master DB user | string | `` | no |
| db_options | A list of DB options to apply with an option group. Depends on DB engine | object | `<list>` | no |
| db_parameter | A list of DB parameters to apply. Note that parameters may differ from a DB family to another | object | `<list>` | no |
| db_parameter_group | Parameter group, depends on DB engine used | string | - | yes |
| deletion_protection | Set to true to enable deletion protection on the RDS instance | bool | `false` | no |
| delimiter | Delimiter to be used between `namespace`, `environment`, `stage`, `name` and `attributes` | string | `-` | no |
| dns_zone_id | The ID of the DNS Zone in Route53 where a new DNS record will be created for the DB host name | string | `` | no |
| enabled | Set to false to prevent the module from creating any resources | bool | `true` | no |
| enabled_cloudwatch_logs_exports | List of log types to enable for exporting to CloudWatch logs. If omitted, no logs will be exported. Valid values (depending on engine): alert, audit, error, general, listener, slowquery, trace, postgresql (PostgreSQL), upgrade (PostgreSQL). | list(string) | `<list>` | no |
| engine | Database engine type | string | - | yes |
| engine_version | Database engine version, depends on engine type | string | - | yes |
| environment | Environment, e.g. 'prod', 'staging', 'dev', 'pre-prod', 'UAT' | string | `` | no |
| final_snapshot_identifier | Final snapshot identifier e.g.: some-db-final-snapshot-2019-06-26-06-05 | string | `` | no |
| host_name | The DB host name created in Route53 | string | `db` | no |
| instance_class | Class of RDS instance | string | - | yes |
| iops | The amount of provisioned IOPS. Setting this implies a storage_type of 'io1'. Default is 0 if rds storage type is not 'io1' | number | `0` | no |
| kms_key_arn | The ARN of the existing KMS key to encrypt storage | string | `` | no |
| license_model | License model for this DB. Optional, but required for some DB Engines. Valid values: license-included | bring-your-own-license | general-public-license | string | `` | no |
| maintenance_window | The window to perform maintenance in. Syntax: 'ddd:hh24:mi-ddd:hh24:mi' UTC | string | `Mon:03:00-Mon:04:00` | no |
| major_engine_version | Database MAJOR engine version, depends on engine type | string | `` | no |
| max_allocated_storage | The upper limit to which RDS can automatically scale the storage in GBs | number | `0` | no |
| multi_az | Set to true if multi AZ deployment must be supported | bool | `false` | no |
| name | Solution name, e.g. 'app' or 'jenkins' | string | `` | no |
| namespace | Namespace, which could be your organization name or abbreviation, e.g. 'eg' or 'cp' | string | `` | no |
| option_group_name | Name of the DB option group to associate | string | `` | no |
| parameter_group_name | Name of the DB parameter group to associate | string | `` | no |
| performance_insights_enabled | Specifies whether Performance Insights are enabled. | bool | `false` | no |
| performance_insights_kms_key_id | The ARN for the KMS key to encrypt Performance Insights data. Once KMS key is set, it can never be changed. | string | `null` | no |
| performance_insights_retention_period | The amount of time in days to retain Performance Insights data. Either 7 (7 days) or 731 (2 years). | number | `7` | no |
| publicly_accessible | Determines if database can be publicly available (NOT recommended) | bool | `false` | no |
| security_group_ids | The IDs of the security groups from which to allow `ingress` traffic to the DB instance | list(string) | `<list>` | no |
| skip_final_snapshot | If true (default), no snapshot will be made before deleting DB | bool | `true` | no |
| snapshot_identifier | Snapshot identifier e.g: rds:production-2019-06-26-06-05. If specified, the module create cluster from the snapshot | string | `` | no |
| stage | Stage, e.g. 'prod', 'staging', 'dev', OR 'source', 'build', 'test', 'deploy', 'release' | string | `` | no |
| storage_encrypted | (Optional) Specifies whether the DB instance is encrypted. The default is false if not specified | bool | `false` | no |
| storage_type | One of 'standard' (magnetic), 'gp2' (general purpose SSD), or 'io1' (provisioned IOPS SSD) | string | `standard` | no |
| subnet_ids | List of subnets for the DB | list(string) | - | yes |
| tags | Additional tags (e.g. `map('BusinessUnit','XYZ')` | map(string) | `<map>` | no |
| vpc_id | VPC ID the DB instance will be created in | string | - | yes |

## Outputs

| Name | Description |
|------|-------------|
| hostname | DNS host name of the instance |
| instance_address | Address of the instance |
| instance_arn | ARN of the instance |
| instance_endpoint | DNS Endpoint of the instance |
| instance_id | ID of the instance |
| option_group_id | ID of the Option Group |
| parameter_group_id | ID of the Parameter Group |
| security_group_id | ID of the Security Group |
| subnet_group_id | ID of the Subnet Group |





## Copyright

Copyright © 2020 [Ceibo IT](https://ceibo.it/copyright)



## License 

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) 

See [LICENSE](LICENSE) for full details.

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.




## Trademarks

All other trademarks referenced herein are the property of their respective owners.



## NOTICE

terraform-aws-rds
Copyright 2020 Ceibo


This product includes software developed by
Cloud Posse, LLC (c) (https://cloudposse.com/)
Licensed under Apache License, Version 2.0
Copyright © 2017-2019 Cloud Posse, LLC
