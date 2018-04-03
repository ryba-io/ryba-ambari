
# Note About modifications brought by Ambari Compliant Install

- HDP 2.6.4.0
- AMBARI 2.6.1

## Ambari Stack repositories Behavior Changes

Ambari [has behavioral change](https://docs.hortonworks.com/HDPDocuments/Ambari-2.6.0.0/bk_ambari-release-notes/content/ambari_relnotes-2.6.0.0-behavioral-changes.html)
about repositories management. It's for better manage HDP,
HDF and other repositories for installing services.
Now Ambari can manage repositories from a single rest entrypoint:
  ```
    curl -v -k -u admin:admin123 -H "X-Requested-By:ambari" -X POST https://master01.metal.ryba:8442/api/v1/version_definitions -d '{
       "VersionDefinition": {
         "version_url": "http://10.10.10.1:10080/centos7/hdp_2.6.4.0/HDP-2.6.4.0-91.xml"
       }
    }'
  ```
  and the response is like
  ```
  "resources" : [
  {
    "href" : "https://master01.metal.ryba:8442/api/v1/version_definitions/1",
    "VersionDefinition" : {
      "id" : 1,
      "stack_name" : "HDP",
      "stack_version" : "2.6"
    }
  }
  ```

Once the id retrieved, services can be add using the `id` to set the stacks you want to use
for installing the service

## Ranger Admin SSL

Now Ranger admin accepts properties towards SSL
  - ranger-admin-site:
    * `ranger.truststore.file`
    * `ranger.truststore.password`

Ranger needs CHARSET set 'Latin' for keys length in mysql setup
  ```
    create database ranger CHARACTER SET=latin1; 
  ```

## JournalNode SSL

Using different principals for JN's principal and snpego
  - hdfs-site:
    * `dfs.journalnode.kerberos.internal.spnego.principal`: `HTTP/_HOST@HADOOP.RYBA`
    * `dfs.journalnode.kerberos.principal`: `jn/_HOST@HADOOP.RYBA`
    * `dfs.journalnode.keytab.file`: `/etc/security/keytabs/jn.service.keytab`

## Remarkable Properties

  - kerberos-env:
    * `manage_identities`: let ambari to manage principals and keytabs of the services
  - ranger-env:
    * `create_db_dbuser`: let ambari create ranger users
  - ranger-admin-site:
    * `ranger.jpa.jdbc.credential.alias`
    * `ranger.truststore.alias`
    Do not mistake with these two properties as they should be different, otherwise
    when trying to connect to the database, ranger will have the wrong password (ie 
    it will use the truststore password).
    To check the password stored execute following commands   
    ```bash
      cd /usr/hdp/current/ranger-admin
      java -cp "cred/lib/*" org.apache.ranger.credentialapi.buildks get "aliasname" -provider jceks://file/etc/ranger/admin/rangeradmin.jceks
    ```
    To get the list of alias stored
    ```bash
      cd /usr/hdp/current/ranger-admin
      java -cp "cred/lib/*" org.apache.ranger.credentialapi.buildks list -provider jceks://file/etc/ranger/admin/rangeradmin.jceks
    ```
    - ranger-hdfs-security:
      * `xasecure.add-hadoop-authorization`: true
      by default, if this property is not present, fallback to hadoop_acl are disabled

## Missing/Required Properties leading to unwanted behavior

    - yarn-site:
      * `yarn.resourcemanager.ha.id`
        if set this property will be used by resourcemanager to get the hostnamen based on
        `yarn.resourcemanager.myid.hostname` and the `_HOST` variable.
    
## Properties needed by Ambari 

options.yarn_site["yarn.resourcemanager.hostname#{id}"] ?= srv.node.fqdn
'yarn.log.server.web-service.url'

## Properties hardcoded in Ambari
These properties change in ryba/ambari toward ryba because they are hardcoded inside
ambari's kerberization scripts.

    - hbase-site:
    * `zookeeper.znode.parent`
      set in ryba to /hbase but hard-coded to /hbase-secure (on hbase_master host)
      It prevents client to open hbase shell

## Ambari And GPL packages
Starting from Ambari 2.6 and HDP 2.6.4, software which are in GPL licensed, has been
moved in their own repositories named `HDP-GPL` and are no more in HDP-UTILS 22
Softwares like:

  - hadooplzo
  - extJS Javascript Library

## Solr and lucidwork-hdpsearch
[Starting from Ambari 2.5](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.4/bk_solr-search-installation/content/ch_hdp-search-install-ambari.html)
now solr is available as a separate mpack and is no more in HDP-utils (HDP-UTILS 22)
It must be installed and registered by ambari-server

  - Download solr mpacks
  - ambari-server install mpacks

## Remarkable Dependencies
  - `HIVE_CLIENT` in `HCAT`
  Because of how ambari calls the hive schemaTool, it needs HIVE_CLIENT to make
  the `/etc/hive/conf` not be empty, as the schemaTools point to this directory by default.
  Ambari uses
  ``` export HIVE_CONF_DIR='/etc/hive/conf.server'
  ```
  but it doesn't work. As a consequence, the /etc/hive/conf should not ne empty
  to make it work

## Interesting links on Ranger Setup and Credentials

## Ambari Background operations orchestrations
