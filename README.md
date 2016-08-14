#### pglogical ansible playbook

Playbook set up postgresql and pglogical replication of 'city' table from 'app_db', it also set access permissions for nodes and application server in pg_hba.conf.

Tested on ubuntu 14.04, with master and slave postgresql 9.5. \
Have no success with 9.4 master and 9.5 slave, because of this error:
```LOG:  starting apply for subscription subscription1
postgres@app_db ERROR:  could not open extension control file "/usr/share/postgresql/9.5/extension/pglogical_origin.control": No such file or directory
postgres@app_db STATEMENT:  CREATE EXTENSION IF NOT EXISTS pglogical_origin WITH SCHEMA pglogical_origin;
pg_restore: [archiver (db)] Error while PROCESSING TOC:
pg_restore: [archiver (db)] Error from TOC entry 2; 3079 16427 EXTENSION pglogical_origin
pg_restore: [archiver (db)] could not execute query: ERROR:  could not open extension control file "/usr/share/postgresql/9.5/extension/pglogical_origin.control": No such file or directory
Command was: CREATE EXTENSION IF NOT EXISTS pglogical_origin WITH SCHEMA pglogical_origin;
```
(pglogical extension cannot be created on 9.4 master without pglogical_origin extension, and after i try to start subscription from 9.5 slave it throws error that pglogical_origin missing and it cannot be installed on 9.5)

Set correct variable values in hosts file before running: \
pg_version - postgresql server version \
pg_master - master (provider) node ip address \
pg_slave - slave (subscriber) node ip address \
pg_port - postgresql server port \
pg_appaddr - ip from which application will access db servers (trust access to 'app_db' for 'postgres' user) \
You also may want to add your public key(s) to roles/ssh_keys/vars/main.yml for passwordless root access.

Add root ssh public key for passwordless access (separate playbook):
```
ansible-playbook -i hosts -u user -k -K root_ssh.yml
```
Setup postgresql with replication:
```
ansible-playbook -i hosts pg_cluster.yml
```
To get lag (after replication is up):
```
ansible-playbook -i hosts pg_cluster.yml --start-at-task="get replication lag"
```

