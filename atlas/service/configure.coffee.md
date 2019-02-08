
# Atlas Master Configuration

    module.exports = (service) ->
      options = service.options

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## Identities

      options.hadoop_group = merge {}, service.deps.hadoop_core.options.hadoop_group, options.hadoop_group
      # Group
      options.group = name: options.group if typeof options.group is 'string'
      options.group ?= {}
      options.group.name ?= 'atlas'
      options.group.system ?= true
      # User
      options.user = name: options.user if typeof options.user is 'string'
      options.user ?= {}
      options.user.name ?= 'atlas'
      options.user.system ?= true
      options.user.comment ?= 'Atlas User'
      options.user.home ?= '/var/lib/atlas'
      options.user.groups ?= ['hadoop']
      options.user.gid = options.group.name

      # Kerberos Hbase Admin Principal
      # options.admin ?= {}
      # options.admin.name ?= options.user.name
      # options.admin.principal ?= "#{options.admin.name}@#{options.krb5.realm}"
      # options.admin.keytab ?= "/etc/security/keytabs/hbase.headless.keytab"
      # throw Error 'Required Option: admin.password' unless options.admin.password

## Kerberos

      # Kerberos HDFS Admin
      options.hdfs_krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user

## Environment

      # Layout
      options.conf_dir ?= '/etc/atlas/conf'
      options.log_dir ?= '/var/log/atlas'
      options.pid_dir ?= '/var/run/atlas'
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
      options.configurations['application-properties'] ?= {}
      options.configurations['application-properties']['atlas.authentication.method.file'] ?= 'false'
      #custom properties
      # atlas.graph.index.search.solr.zookeeper-url
      # atlas.rest.address      

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
