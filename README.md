# MySQL

## Introduction

MySQL is an open source relational database management system. AppZ currently offers MySQL-5.7.

### Features:

1. Fully managed stateful MySQL-5.7 deployment using GitOps from Client’s Git repository.
1. All the Database secrets are  Stored  in AppZ Vault.
1. Database Changes(Creation of New database, user, schema, tables etc and restoring the database) deployed using GitOps.
1. Built-in logging and monitoring visualization through AppZ Dashboard.

## Pre-requisites
* **Setup AppZ Cluster**- Install **AppZ** Cluster from AWS Marketplace. Find the instructions at our [product documentation](https://docs.ecloudcontrol.com/installer-3.0/aws-marketplace/).
Before deploying MySQL application, make sure the [Vault](https://docs.ecloudcontrol.com/vault-1.2/) application is deployed.


## Sample Project

[MySQL](https://github.com/ecloudcontrol/MySQL) - Use this sample project to deploy MySQL-5.7.

If you fork [MySQL](https://github.com/ecloudcontrol/MySQL) under your Git account, you will see the following `appz.yml` and `setup.yaml` in the root of your project.

### appz.yaml

```
    app:
        name: MySQL
        code: mysql
        notify: appzdev@cloudbourne.co

    build:
        version: 5.7
        env: DEV
        build_file: none
        output_files: output/*.zip
        image_template: mysql-5.7

    deploy:
        context: alpha/DEV
        type: statefulset
        replicas: 1
        port:
        - 3306

    volumes:
      - claim: mysql-data
        mount: /var/lib/mysql
        name: data
        size: 5Gi
      - claim: mysql-backup
        mount: /appz/backup
        name: backup
        size: 5Gi

    properties:
        MYSQL_ROOT_PASSWORD:
          vault: MYSQL_ROOT_PASSWORD_KEY
        MYSQL_SPRINGBOOTWEB_PASSWORD:
          vault: MYSQL_SPRINGBOOTWEB_PASSWORD
        revision : 24
        nano: 67
```

### Properties

The image is vault enabled so secrets can be pulled from vault. These are placed under `properties` in `appz.yml`. Please check each one below, with its actual purpose.

- **MYSQL_SPRINGBOOTWEB_PASSWORD** - Password of the MySQL user 'springboot-web'.
- **MYSQL_ROOT_PASSWORD** - Password of the MySQL user 'root'.

###  setup.yaml

```
---

users:
  - name: springboot-web
    password: $MYSQL_SPRINGBOOTWEB_PASSWORD

databases:
  - name: springboot_web

acl:
  - database: springboot_web
    user: springboot-web
    access: ALL

restore:
  - database: springboot_web
    source:
       url: 'https://www.ecloudcontrol.com/wp-content/uploads/2020/12/accountdb_20201210.zip'
    user: root
    password: $MYSQL_ROOT_PASSWORD  # password of the 'user' above.
    token:  20201211-1415 # The restore token to be validated before restoring the database. (Same as TEARDOWN_TOKEN)
```


 * In the users section, name of each user (to be created) with password is required.

 * In the databases section, details of each database (to be created) is required.

 * In the acl section, details of the database, username and the privilege to be granted is required.

 * In the restore section, the database name, the restore token and the backup url should be given. For password protected urls, authorised username and password should be given (under the 'source') in addition to these.


### Resilience

Capable of running unlimited number of replicas across multiple regions/datacenters.

### Volumes

[Standard AppZ Volumes](../volumes.md) are enabled.

* **/var/lib/mysql** - MySQL data folder - Mandatory

* **/appz/backup** - Backup of all databases will be saved once a day
