ansible-aws-rds
=========

* Create RDS subnet group. 
  Two private subnets for RDS have to be created in advance, which have a tag named RDSName with value of {{env}}_rds_private1/2.
* Create RDS paramaters group. 
  Refer to the vars required in the Example section below.
* Creates RDS instances.
  The aws.group_facts.yml will get vpc security group id, you just need to define the name. 
* Restore database, by restore.yml. 
** Added condition to main.yml to call different task file. Use command: restore or create as condition when calling this role

Requirements
------------

* boto
* boto3 for aws.group_facts.yml 

Role Variables
--------------

Role accepts list of servers to create. Each list is a dictionary with required parameters.

Dependencies
------------

none

Example Playbook
----------------

    - hosts: databases
      vars:
      aws_rds                 :
        - security_group_id   : "sg-3a27b45z"
          publicly_accessible : False
          db_engine           : "postgres"
          db_name             : "testdb"
          instance_name       : "testremoveme"
          engine_version      : "9.4.5"
          size                : "20"
          instance_type       : "db.t2.micro"
          wait                : true
          wait_timeout        : 600
          command             : "create"
          username            : "abcdefg"
          password            : "abcdefg123"
        # vars required for subnetgroup and parameter group added: 
          vpc_subnet              : "RDS-{{ env }}-subnetgroup"
          para_grp                : "RDS-{{ env }}-parametergroup"
          paragrp_engine          : "mysql5.6"
          rds_parameters          :
              - { param: 'max_heap_table_size', value: '33554432' }
              - { param: 'sort_buffer_size', value: '16777216' }
              - { param: 'tmp_table_size', value: '33554432' }

      roles:
         - { role: ansible-aws-rds }


