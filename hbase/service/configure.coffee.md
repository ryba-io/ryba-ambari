
# HBase Master Configuration

    module.exports = (service) ->
      options = service.options

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## Identities

* `admin` (object|string)
  The Kerberos HBase principal.
* `group` (object|string)
  The Unix HBase group name or a group object (see Nikita Group documentation).
* `user` (object|string)
  The Unix HBase login name or a user object (see Nikita User documentation).

Example

```json
{
  "user": {
    "name": "hbase", "system": true, "gid": "hbase", groups: "hadoop",
    "comment": "HBase User", "home": "/var/run/hbase"
  },
  "group": {
    "name": "HBase", "system": true
  },
  "admin": {
    "password": "hbase123"
  }
}
```

      # Hadoop Group
      options.hadoop_group = merge {}, service.deps.hadoop_core.options.hadoop_group, options.hadoop_group
      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'hbase'
      options.group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'hbase'
      options.user.system ?= true
      options.user.gid = options.group.name
      options.user.comment ?= 'HBase User'
      options.user.home ?= '/var/run/hbase'
      options.user.groups ?= 'hadoop'
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 32000
      # Kerberos Hbase Admin Principal
      options.admin ?= {}
      options.admin.name ?= options.user.name
      options.admin.principal ?= "#{options.admin.name}@#{options.krb5.realm}"
      options.admin.keytab ?= "/etc/security/keytabs/hbase.headless.keytab"
      throw Error 'Required Option: admin.password' unless options.admin.password

## Kerberos

      # Kerberos HDFS Admin
      options.hdfs_krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user

## Environment

      # Layout
      options.conf_dir ?= '/etc/hbase/conf'
      options.log_dir ?= '/var/log/hbase'
      options.pid_dir ?= '/var/run/hbase'
      options.hbase_tmp ?= '/var/log/tmp'
      # Env
      options.env ?= {}
      options.env['HBASE_LOG_DIR'] ?= "#{options.log_dir}"
      options.env['HBASE_OPTS'] ?= '-XX:+UseConcMarkSweepGC ' # -XX:+CMSIncrementalMode is deprecated
      # Java
      # 'HBASE_MASTER_OPTS' ?= '-Xmx2048m' # Default in HDP companion file
      options.java_home ?= "#{service.deps.java.options.java_home}"
      ## System Options
      options.opts ?= {}
      options.opts.base ?= ''
      options.opts.java_properties ?= {}
      options.opts.jvm ?= {}

      # Misc
      options.fqdn ?= service.node.fqdn
      options.hostname = service.node.hostname
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.clean_logs ?= false
      # HDFS
      options.hdfs_conf_dir ?= service.deps.hadoop_core.options.conf_dir
      options.hdfs_krb5_user ?= service.deps.hadoop_core.options.hdfs.krb5_user

## Ambari Configuration

      options.configurations ?= {}
      options.configurations['hbase-site'] ?= {}
      # Env
      options.configurations['hbase-env'] ?= {}
      #master
      options.configurations['hbase-env']['hbase_master_heapsize'] ?= options.heapsize

## HBase Policy

      options.configurations['hbase-policy'] ?= {}
      options.configurations['security.client.protocol.acl'] ?= '*'
      options.configurations['security.admin.protocol.acl'] ?= '*'
      options.configurations['security.masterregion.protocol.acl'] ?= '*'

## Ambari HBase Hosts

      options.regionserver_hosts ?= []
      options.master_hosts ?= []
      options.client_hosts ?= []
      options.rest_hosts ?= []
      options.thrift_hosts ?= []

## Ambari REST API

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name

      options.configurations ?= {}
      options.configurations['hbase-site'] ?= {}
      options.configurations['hbase-site']['hbase.tmp.dir'] ?= "#{options.log_dir}/tmp"

## System Options
      
      options.master_opts ?= {}
      options.regionserver_opts ?= {}
      # Env
      options.configurations['hbase-env'] ?= {}
      options.configurations['hbase-env']['hbase_user'] ?= options.user.name
      options.configurations['hbase-env']['hbase_principal_name'] ?= options.admin.principal
      options.configurations['hbase-env']['hbase_user_keytab'] ?= options.admin.keytab
      options.configurations['hbase-env']['hbase_user_nofile_limit'] ?= options.user.limits.nofile
      options.configurations['hbase-env']['hbase_user_nproc_limit'] ?= options.user.limits.nproc
      options.configurations['hbase-env']['hbase_log_dir'] ?= options.log_dir
      options.configurations['hbase-env']['hbase_pid_dir'] ?= options.pid_dir
      options.configurations['hbase-env']['hbase_tmp_dir'] ?= "#{options.log_dir}/tmp"
      options.configurations['hbase-env']['hbase_regionserver_xmn_ratio'] ?= '0.2'
      options.configurations['hbase-env']['hbase_regionserver_xmn_max'] ?= '512'
      options.configurations['hbase-env']['hbase_java_io_tmpdir'] ?= options.configurations['hbase-env']['hbase_tmp_dir']
      options.configurations['hbase-env']['java_home'] ?= options.java_home
      options.configurations['hbase-env']['java_home64'] ?= options.java_home

## Log4j Properties

      options.hbase_log4j ?= {}

## Ambari Agent
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['hbase'] ?= options.user
        srv.options.groups ?= {}
        srv.options.groups['hbase'] ?= options.group

## Dependencies

    appender = require 'ryba/lib/appender'
    {merge} = require 'nikita/lib/misc'

## Resources

*   [Tuning G1GC For Your HBase Cluster](https://blogs.apache.org/hbase/entry/tuning_g1gc_for_your_hbase)
*   [HBase: Performance Tunners (read optimization)](http://labs.ericsson.com/blog/hbase-performance-tuners)
*   [Scanning in HBase (read optimization)](http://hadoop-hbase.blogspot.com/2012/01/scanning-in-hbase.html)
*   [Configuring HBase Memstore (write optimization)](http://blog.sematext.com/2012/17/16/hbase-memstore-what-you-should-know/)
*   [Visualizing HBase Flushes and Compactions (write optimization)](http://www.ngdata.com/visiualizing-hbase-flushes-and-compactions/)

[SecureBulkLoadEndpoint]: http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/security/access/SecureBulkLoadEndpoint.html
