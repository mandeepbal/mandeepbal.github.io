---
layout: post
title: Deploying ManageIQ (Anand Release)
---

```
chmod 777 /opt
useradd miqbuilder
echo "" >> /etc/sudoers
echo "" >> /etc/sudoers
echo "miqbuilder ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
yum -y install http://dl.fedoraproject.org/pub/epel/6Server/x86_64/epel-release-6-8.noarch.rpm
iptables -I INPUT -p tcp --dport 3000 -j ACCEPT
service iptables save
service iptables restart
yum -y install git libxml2-devel libxslt libxslt-devel sudo
yum -y install postgresql-server postgresql-devel memcached ruby-devel
service memcached start
chkconfig memcached on
chkconfig postgresql on
service postgresql initdb
echo "local all all trust" > /var/lib/pgsql/data/pg_hba.conf
echo "" >> /var/lib/pgsql/data/postgresql.conf
echo "listen_addresses = '*'" >> /var/lib/pgsql/data/postgresql.conf
service postgresql start

##Run as Postgres user
su - postgres
for i in test production development;do createdb vmdb_$i;done
psql -c "create role root login password 'smartvm'"
psql -c "alter database vmdb_development owner to root"
exit

##Run as MIQ user
su - miqbuilder
\curl -sSL https://get.rvm.io | bash
. ~/.bash_profile
rvm install 1.9.3
gem uninstall bundler
gem uninstall -i /home/miqbuilder/.rvm/gems/ruby-1.9.3-p551@global bundler

##Bundler should be uninstalled
gem install bundler -v '1.3.5'
gem install ruby-graphviz
cd /opt
git clone https://github.com/manageiq/manageiq.git
cd manageiq
git checkout anand-1
cd vmdb
bundle install --without qpid
cd ..
vmdb/bin/rake build:shared_objects
cd vmdb
bundle install --without qpid
cp config/database.pg.yml config/database.yml
bin/rake db:migrate
cd /opt/manageiq/vmdb/
bin/rake evm:start
```
