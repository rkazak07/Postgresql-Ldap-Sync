# Postgresql Ldap Sync

### Overview
With this Application, you can provide automatic ldap sync to your postgresql architectures installed on operating systems such as RedHat, Centos, Rocky linux, Oracle Linux.

### How to use

* Configurable per YAML config file
* Can use Active Directory as LDAP-Server
* Nested groups/roles supported
* Set scope of considered users/groups on LDAP and PG side
* Test mode which doesn't do any changes to the DBMS
* Both LDAP and PG connections can be secured by SSL/TLS

### Requirements

* Ruby-2.0+, JRuby-1.2+
* LDAP-v3 server
* PostgreSQL-server v9.0+
* Oracle-Linux/Centos/Rhel/RockyLinux 

### Installation

```bash
  $ yum install -y ruby rubygem-rake rubygems ruby-devel openldap-clients git wget tar curl make  rubygem-bigdecimal.x86_64 redhat-rpm-config libpq-devel.x86_64 gcc nano
  ```
  ```
  $ git clone https://github.com/rkazak07/Postgresql-ldap-sync.git
  ```
  ```bash
  $ cd Postgresql-ldap-sync
  ```
  ```bash
  $ gem install bundler
  ```
  ```bash
  $ bundle install
  ```
  ```bash
  $ bundle exec rake install
  ```
  ```bash
  $ gem install json
  ```
  ```bash
  $ which pg_ldap_sync
  ```


## Postgresql
We add roles for groups and users in Postgresql.
 ```bash
  $ sudo -su postgres psql
 ```
 ```bash
  $ create role ldap_users;
   ```
 ```bash
  $ create role ldap_groups;
   ```
 ```bash
   \du
 ```
 
 ## Usage

Create a config file based on
[config/pg-ldap-sync-config.yaml](https://github.com/rkazak07/Postgresql-ldap-sync/config/pg-ldap-sync.yaml)

Run in test-mode:
```bash
  $ pg_ldap_sync -c my_config.yaml -vv -t
```
Run in modify-mode:
```bash
  $ pg_ldap_sync -c my_config.yaml -vv
```
 
 
 ## Check
 Check whether the users taken from the active directory are written to Postgresql. If users appear in roles when you run the below command, they have been successfully added.
 ```bash
 $ sudo -su postgres psql
 ```
 ```bash
  \du
 ```
 
### Ldap Test
 ```bash
 $ ldapsearch -x -h ad-host-ip -D "pgadsync@domain.local" -W "(sAMAccountName=*)" -b "OU=pgusers,OU=Service_Users,OU=organization-unit,DC=domain,DC=local"  | grep    sAMAccountName
 ```
> Ldap Example
> 
#filter: (sAMAccountName=*)
sAMAccountName: user1
sAMAccountName: user2


### POSTGRESQL
> Postgresql pg_hba.conf add  ldap sync parameters 
```sh
$ nano /var/lib/pgsql/13/data/pg_hba.conf
```
host    all             all             0.0.0.0/0               ldap ldapserver=domain-host ldapport=389 ldapprefix=""  ldapsuffix="@domain.local" ldapscheme=ldap
```
$ systemctl restart postgresql-14
```

> Now we are setting the user that will create the roles and authorizations between postgresql' and AD from the users we have added to the database.
> 
```bash
  $ sudo -su postgres psql
```
```bash
$ create role "user1" superuser createdb createrole;
 ```
> If we are going to create a user from existing ones, we are changing its authority
 ```bash
$ alter role "user1" superuser createdb createrole;
 ```
> We will authorize the pggroup we created via AD to postgres.
 
 ```bash
 $ drop role pggroup;
 ```
 ```bash
 $ create role pggroup in role ldap_groups;
 ```
 ```bash
 $ grant CONNECT ON DATABASE postgres to pggroup;
 ```
 
> Postgresql Database Ldap Login Control
  ```bash
 $ psql -h db-host-ip -U "ldapuser" -d postgres
 ```

### CRONJOB:
> We need to create a cronjob so that the pg-ldap-sync.yaml file we created can pull the users added to the pggroup via AD in certain periods.
> 
```bash
 $ sudo yum -y install crontabs
 crontab -e
 ```
 We specify pg-ldap-sync and its runtime to the crontab.
 ```bash
 $ crontab -e
 ```
 
