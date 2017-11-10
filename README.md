ansible-aws-rds
=========

* Create RDS subnet group with 2 existing private subnets.
* Create RDS paramaters group.
* Creates RDS instances.
* Restore databases.
* Replicate RDS to default subnet(this is due to ansible can specify subnetgroup only when creating a RDS instance)
* Gather facts of defined rds instance


Requirements
------------

* boto
* boto3 for aws.group_facts.yml


Role Variables
--------------

Role accepts list of servers to create. Each list is a dictionary with required parameters.
* security_group_id is replaced with vpc_security_groups
* To create cross region rds replica, [define source_instance var with ARN](http://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance-read-replica.html)
* Added condition to main.yml to call different task files. Pass command option to this role, or it will only gather facts by default.

Dependencies
------------

* Two private subnets for RDS have to be created in advance, which have a tag named RDSName with value of {{env}}_rds_private1/2.
* A security group for RDS has to be created in advance, and defined for aws.group_facts.yml to get it's vpc security group id.

Example Playbook
----------------
It listed all different ways to call this role, choose one of them.

    - hosts: databases
      vars:
      aws_rds                 :
        - publicly_accessible : False
          db_engine           : "MySQL"
          db_name             : "testdb"
          instance_name       : "testremoveme"
          engine_version      : "5.6.34"
          size                : "20"
          instance_type       : "db.t2.micro"
          wait                : true
          wait_timeout        : 600
          username            : "abcdefg"
          password            : "abcdefg123"
        # vars to modify rds for parameters not available with create/restore/replica
          multi_zone          : "yes"
          apply_immediately   : "yes"
          backup_retention        : 0
          upgrade                 : yes
          port                    : "3306"
          region                  : "{{ec2_region}}"
        # vars for restore, this snapshot has to be exist in this region.
          snapshot            : "rdsbackp-2017-10-30-08-10"
        # vars for replicate, in case of cross region replicate, put ARN here.
          source_instance     : "{{ type }}-{{ org }}-{{ env }}-sourceinst"        
        # vars for securitygroup/subnetgroup and parametergroup
          vpc_security_groups     : "RDS-{{ env }}-securitygroup"
          vpc_subnet              : "RDS-{{ env }}-subnetgroup"
          para_grp                : "RDS-{{ env }}-parametergroup"
          paragrp_engine          : "mysql5.6"
          rds_parameters          :
              - { param: 'max_heap_table_size', value: '33554432' }
              - { param: 'sort_buffer_size', value: '16777216' }
              - { param: 'tmp_table_size', value: '33554432' }
        # optional
          instance_tags            :
              Name: '{{type}}_{{env}}'
              Type: '{{type}}'
              Organization: '{{org}}'
              Environment: '{{env}}'
              Role: '{{env}}-internal_system'

      roles:
         - { role: ansible-aws-rds } # Gather facts of this rds instance only.
         - { role: ansible-aws-rds, command: create}
         - { role: ansible-aws-rds, command: restore}
         - { role: ansible-aws-rds, command: replicate}
