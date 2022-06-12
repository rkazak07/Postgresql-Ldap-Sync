## FEATURES:

* Configurable per YAML config file
* Can use Active Directory as LDAP-Server
* Nested groups/roles supported
* Set scope of considered users/groups on LDAP and PG side
* Test mode which doesn't do any changes to the DBMS
* Both LDAP and PG connections can be secured by SSL/TLS

## REQUIREMENTS:

* Ruby-2.0+, JRuby-1.2+
* LDAP-v3 server
* PostgreSQL-server v9.0+

## INSTALL:

Install Ruby:

* on Oracle-Linux8/Centos8/Rhel8/RockyLinux8: 

### Install from Git:
```sh
  yum install -y ruby rubygem-rake rubygems ruby-devel openldap-clients git wget tar curl make  rubygem-bigdecimal.x86_64 redhat-rpm-config libpq-devel.x86_64 gcc nano
  git clone https://github.com/larskanis/pg-ldap-sync.git
  cd pg-ldap-sync
  gem install bundler
  bundle install
  bundle exec rake install
  gem install json
  which pg_ldap_sync
```


## POSTGRESQL13:
We add roles for groups and users in Postgresql.
```sh
  su postgres
 ```
  ```sh
   psql
   ```
   ```sh
   create role ldap_users;
   ```
   ```sh
   create role ldap_users;
   ```
   ```sh
  \du
 ```
 
 ## USAGE:

Create a config file based on
[config/pg-ldap-sync-config.yaml](https://github.com/rkazak07/Oracle-Linux8-Postgresql-13-ldap-sync/config/pg-ldap-sync.yaml)

Run in test-mode:
```sh
  pg_ldap_sync -c my_config.yaml -vv -t
```
Run in modify-mode:
```sh
  pg_ldap_sync -c my_config.yaml -vv
```
 
 
 ## CHECK:
 Check whether the users taken from the active directory are written to Postgresql. If users appear in roles when you run the below command, they have been successfully added.
 ```sh
 su postgres
 ```
 ```sh
 psql
 ```
 ```sh
  \du
 ```
 
 ## LDAP TEST:
 ```sh
 ldapsearch -x -h ad-host-ip -D "pgadsync@domain.local" -W "(sAMAccountName=*)" -b "OU=pgusers,OU=Service_Users,OU=organization-unit,DC=domain,DC=local"  | grep    sAMAccountName
 ```
## EXAMPLE:
```sh
#filter: (sAMAccountName=*)
sAMAccountName: user1
sAMAccountName: user2
```

## PG_HBA.CONF EDIT:
```sh
nano /var/lib/pgsql/13/data/pg_hba.conf
```
```sh
host    all             all             0.0.0.0/0               ldap ldapserver=domain-host-ip ldapport=389 ldapprefix=""
```

## POSTGRESQL13:
Now we are setting the user that will create the roles and authorizations between postgresql' and AD from the users we have added to the database.
```sh
 create role "user1" superuser createdb createrole;
 ```
 If we are going to create a user from existing ones, we are changing its authority
 ```sh
 alter role "user1" superuser createdb createrole;
 ```
 We will authorize the pggroup we created via AD to postgres.
 ```sh
 drop role pggroup; 
 create role pggroup in role ldap_groups;
 grant CONNECT ON DATABASE postgres to pggroup;
 ```

## CRONJOB:
We need to create a cronjob so that the pg-ldap-sync.yaml file we created can pull the users added to the pggroup via AD in certain periods.
# example:
```sh
 sudo yum -y install crontabs
 crontab -e
 ```
 We specify pg-ldap-sync and its runtime to the crontab.
 ```sh
 crontab -e
 ```
 
