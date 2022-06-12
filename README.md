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
Postgresql içerisinde grup ve user'lar için rol ekliyoruz.
```sh
   su postgres
   psql
   create role ldap_users;
   create role ldap_users;
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
 AD üzerinden alınan kullanıcıların Postgresql'e yazılıp yazılmadığını kontrol edelim. Aşağıdkai komutu çalıştırınca user'lar rollerde görünüyorsa başarıyla eklenmiştir.
 ```sh
  su postgres
  psql
  \du
 ```
 
 ## LDAP TESTI:
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
host    all             all             0.0.0.0/0               ldap ldapserver=domain-host-ip ldapport=389 ldapprefix=""

## POSTGRESQL13:
 şimdi db'ye eklediğimiz user'lardan postgresql' ile AD arasında rol ve yetkilendirmeleri oluşturacak olan user'ı ayarlıyoruz.
```sh
 create role "user1" superuser createdb createrole;
 ```
 Eğer user mevcutlardan oluşturacaksak yetkisini değiştiriyoruz.
 ```sh
 alter role "user1" superuser createdb createrole;
 ```
AD üzerinden oluşturduğumuz pggroup'a postgres'e yetki vereceğiz.
 ```sh
 drop role pggroup; 
 create role pggroup in role ldap_groups;
 grant CONNECT ON DATABASE postgres to pggroup;
 ```

## CRONJOB:
Oluşturduğumuz pg-ldap-sync.yaml dosyasını AD üzerinden pggroup'a eklenen userları belirli periyorlarda çekebilmesi için cronjob  oluşturmalıyız.
# example:
```sh
 sudo yum -y install crontabs
 crontab -e
 ```
# Crontab'a pg-ldap-sync'i ve çalıştırma süresini belirtiyoruz.
 ```sh
 crontab -e
 ```
 
