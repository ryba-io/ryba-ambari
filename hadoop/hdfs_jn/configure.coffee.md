
# Hadoop HDFS JournalNode Configure

The JournalNode uses properties define inside the "ryba-ambari-takeover/hadoop/hdfs" module. It
also declare a new property "dfs.journalnode.edits.dir".

*   `hdp.hdfs.site['dfs.journalnode.edits.dir']` (string)
    The directory where the JournalNode will write transaction logs, default
    to "/var/run/hadoop-hdfs/journalnode\_edit\_dir"

Example:

```json
{
  "site": {
    "dfs.journalnode.edits.dir": "/var/run/hadoop-hdfs/journalnode\_edit\_dir"
  }
}
```

    module.exports = (service) ->
      options = service.options
      options.configurations ?= {}

## Identities

      options.group = merge {}, service.deps.hdfs[0].options.hdfs.group, options.group
      options.user = merge {}, service.deps.hdfs[0].options.hdfs.user, options.user
      options.hadoop_group = merge {}, service.deps.hdfs[0].options.hadoop_group, options.hadoop_group

## Environment

      options.pid_dir ?= service.deps.hadoop_core.options.hdfs.pid_dir
      options.log_dir ?= service.deps.hadoop_core.options.hdfs.log_dir
      options.conf_dir ?= '/etc/hadoop-hdfs-journalnode/conf'
      options.hadoop_opts ?= service.deps.hadoop_core.options.hadoop_opts
      # Java
      options.java_home ?= service.deps.java.options.java_home
      options.hadoop_heap ?= service.deps.hadoop_core.options.hadoop_heap
      options.newsize ?= '200m'
      options.heapsize ?= '1024m'
      # Misc
      options.clean_logs ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.fqdn = service.node.fqdn
      options.hdfs_krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user

## Identities

      options.hadoop_group = merge {}, service.deps.hadoop_core.options.hadoop_group, options.hadoop_group
      options.group = merge {}, service.deps.hadoop_core.options.hdfs.group, options.group
      options.user = merge {}, service.deps.hadoop_core.options.hdfs.user, options.user

## System Options

      options.opts ?= {}
      options.opts.base ?= ''
      options.opts.java_properties ?= {}
      options.opts.jvm ?= {}
      # options.opts.jvm['-Xms'] ?= options.heapsize
      # options.opts.jvm['-Xmx'] ?= options.heapsize
      # options.opts.jvm['-XX:NewSize='] ?= options.newsize #should be 1/8 of datanode heapsize
      # options.opts.jvm['-XX:MaxNewSize='] ?= options.newsize #should be 1/8 of datanode heapsize

## Configuration

      options.core_site = merge {}, service.deps.hadoop_core.options.core_site, options.core_site or {}
      options.hdfs_site ?= {}
      options.hdfs_site['dfs.journalnode.rpc-address'] ?= '0.0.0.0:8485'
      options.hdfs_site['dfs.journalnode.http-address'] ?= '0.0.0.0:8480'
      options.hdfs_site['dfs.journalnode.https-address'] ?= '0.0.0.0:8481'
      options.hdfs_site['dfs.http.policy'] ?= 'HTTPS_ONLY'
      # Recommandation is to ideally have dedicated disks to optimize fsyncs operation
      options.hdfs_site['dfs.journalnode.edits.dir'] = options.hdfs_site['dfs.journalnode.edits.dir'].join ',' if Array.isArray options.hdfs_site['dfs.journalnode.edits.dir']
      # options.hdfs_site['dfs.journalnode.edits.dir'] ?= ['/var/hdfs/edits']
      throw Error "Required Option \"hdfs_site['dfs.journalnode.edits.dir']\": got #{JSON.stringify options.hdfs_site['dfs.journalnode.edits.dir']}" unless options.hdfs_site['dfs.journalnode.edits.dir']

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      #takeover config
      # options.hdfs_site['dfs.journalnode.kerberos.internal.spnego.principal'] = "HTTP/_HOST@#{options.krb5.realm }"
      # options.hdfs_site['dfs.journalnode.kerberos.principal'] = "HTTP/_HOST@#{options.krb5.realm }"
      # options.hdfs_site['dfs.journalnode.keytab.file'] = '/etc/security/keytabs/spnego.service.keytab'
      ## start should be config
      # options.hdfs_site['dfs.journalnode.kerberos.internal.spnego.principal'] = "HTTP/_HOST@#{options.krb5.realm }"
      # options.hdfs_site['dfs.journalnode.kerberos.principal'] = "jn/_HOST@#{options.krb5.realm }"
      # options.hdfs_site['dfs.journalnode.keytab.file'] = '/etc/security/keytabs/jn.service.keytab'
      # end should be config

      # options.opts.java_properties['java.security.auth.login.config'] ?= "#{options.conf_dir}/hdfs_jn_jaas.conf"

# ## Ambari Kerberos Principal and Keytab
# 
#       for srv in service.deps.ambari_server
#         srv.options.identities ?= {}
#         srv.options.identities['journalnode_jn'] ?= {}
#         srv.options.identities['journalnode_jn']['principal'] ?= {}
#         srv.options.identities['journalnode_jn']['principal']['configuration'] ?= 'hdfs-site/dfs.journalnode.kerberos.principal'
#         srv.options.identities['journalnode_jn']['principal']['type'] ?= 'service'
#         srv.options.identities['journalnode_jn']['principal']['local_username'] ?= options.user.name
#         srv.options.identities['journalnode_jn']['principal']['value'] ?= options.hdfs_site['dfs.journalnode.kerberos.principal']
#         srv.options.identities['journalnode_jn']['name'] ?= 'journalnode_jn'
#         srv.options.identities['journalnode_jn']['keytab'] ?= {}
#         srv.options.identities['journalnode_jn']['keytab']['owner'] ?= {}
#         srv.options.identities['journalnode_jn']['keytab']['owner']['access'] ?= 'r'
#         srv.options.identities['journalnode_jn']['keytab']['owner']['name'] ?= options.user.name
#         srv.options.identities['journalnode_jn']['keytab']['group'] ?= {}
#         srv.options.identities['journalnode_jn']['keytab']['group']['access'] ?= 'r'
#         srv.options.identities['journalnode_jn']['keytab']['group']['name'] ?= options.hadoop_group.name
#         srv.options.identities['journalnode_jn']['keytab']['file'] ?= options.hdfs_site['dfs.journalnode.kerberos.principal']
#         srv.options.identities['journalnode_jn']['keytab']['configuration'] ?= 'hdfs-site/dfs.journalnode.kerberos.principal'


## SSL

      options.ssl = merge {}, service.deps.hadoop_core.options.ssl, options.ssl
      options.ssl_server = merge {}, service.deps.hadoop_core.options.ssl_server, options.ssl_server or {}
      options.ssl_client = merge {}, service.deps.hadoop_core.options.ssl_client, options.ssl_client or {}

## Wait

      options.wait_krb5_client = service.deps.krb5_client.options.wait
      options.wait_zookeeper_server = service.deps.zookeeper_server[0].options.wait
      options.wait = {}
      options.wait.rpc = for srv in service.deps.hdfs_jn
        srv.options.hdfs_site ?= {}
        srv.options.hdfs_site['dfs.journalnode.rpc-address'] ?= '0.0.0.0:8485'
        [_, port] = srv.options.hdfs_site['dfs.journalnode.rpc-address'].split ':'
        host: srv.node.fqdn, port: port
      options.wait.http = for srv in service.deps.hdfs_jn
        srv.options.hdfs_site ?= {}
        policy = srv.options.hdfs_site['dfs.http.policy'] or options.hdfs_site['dfs.http.policy']
        address = if policy is 'HTTP_ONLY'
        then srv.options.hdfs_site['dfs.journalnode.http-address'] or '0.0.0.0:8480'
        else srv.options.hdfs_site['dfs.journalnode.https-address'] or '0.0.0.0:8481'
        [_, port] = address.split ':'
        host: srv.node.fqdn, port: port
      options.hosts = service.deps.hdfs_jn.map (srv) -> srv.node.fqdn

## Dependencies

    {merge} = require 'nikita/lib/misc'
