
# Hive Server2 Install

TODO: Implement lock for Hive Server2
http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH4/4.2.0/CDH4-Installation-Guide/cdh4ig_topic_18_5.html

HDP 2.1 and 2.2 dont support secured Hive metastore in HA mode, see
[HIVE-9622](https://issues.apache.org/jira/browse/HIVE-9622).

Resources:
*   [Cloudera security instruction for CDH5](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cdh_sg_hiveserver2_security.html)

    module.exports =  header: 'Ambari Hive Server2 Install', handler: (options) ->

## Register

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'

## Wait

      @call once: true, 'ryba-ambari-takeover/hive/hcatalog/wait'

## IPTables

| Service        | Port  | Proto | Parameter            |
|----------------|-------|-------|----------------------|
| Hive Server    | 10001 | tcp   | env[HIVE_PORT]       |


IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      hive_server_port = if options.hive_site['hive.server2.transport.mode'] is 'binary'
      then options.hive_site['hive.server2.thrift.port']
      else options.hive_site['hive.server2.thrift.http.port']
      rules = [{ chain: 'INPUT', jump: 'ACCEPT', dport: hive_server_port, protocol: 'tcp', state: 'NEW', comment: "Hive Server" }]
      rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: parseInt(options.jmx_port,10), protocol: 'tcp', state: 'NEW', comment: "HiveServer2 JMX" } if options.jmx_port?
      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: rules

## Identities

By default, the "hive" and "hive-hcatalog" packages create the following
entries:

```bash
cat /etc/passwd | grep hive
hive:x:493:493:Hive:/var/lib/hive:/sbin/nologin
cat /etc/group | grep hive
hive:x:493:
```

      @system.group header: 'Group', options.group
      @system.user header: 'User', options.user

## Ulimit

      @system.limits
        header: 'Ulimit'
        user: options.user.name
      , options.user.limits

## Startup

Install the "hive-server2" service, symlink the rc.d startup script
inside "/etc/init.d" and activate it on startup.

The server is not activated on startup because they endup as zombies if HDFS
isnt yet started.

      @system.tmpfs
        if_os: name: ['redhat','centos'], version: '7'
        mount: options.pid_dir
        uid: options.user.name
        gid: options.group.name
        perm: '0750'

## Layout

Create the directories to store the logs and pid information. The properties
"ryba.hive.server2.log\_dir" and "ryba.hive.server2.pid\_dir" may be modified.

      @call header: 'Layout', ->
        @system.mkdir
          target: options.log_dir
          uid: options.user.name
          gid: options.group.name
          parent: true
        @system.mkdir
          target: options.pid_dir
          uid: options.user.name
          gid: options.group.name
          parent: true

## SSL

      @call
        header: 'SSL'
        if: -> options.hive_site['hive.server2.use.SSL'] is 'true'
      , ->
        @java.keystore_add
          keystore: options.hive_site['hive.server2.keystore.path']
          storepass: options.hive_site['hive.server2.keystore.password']
          key: options.ssl.key.source
          cert: options.ssl.cert.source
          keypass: options.hive_site['hive.server2.keystore.password']
          name: options.ssl.key.name
          local: options.ssl.key.local
        @java.keystore_add
          keystore: options.hive_site['hive.server2.keystore.path']
          storepass: options.hive_site['hive.server2.keystore.password']
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local

## Kerberos

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos'
        unless: options.principal_identical_to_hcatalog
        principal: options.hive_site['hive.server2.authentication.kerberos.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.hive_site['hive.server2.authentication.kerberos.keytab']
        uid: options.user.name
        gid: options.group.name

## Wait TEZ Service

      @ambari.services.wait
        header: 'TEZ Service WAITED'
        if: options.hive_site['hive.execution.engine'] is 'tez'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'TEZ'

## Install Component

      @ambari.hosts.component_wait
        header: 'HIVE_SERVER'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_SERVER'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'HIVE_SERVER'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_SERVER'
        hostname: options.fqdn

## Dependencies

    path = require 'path'
