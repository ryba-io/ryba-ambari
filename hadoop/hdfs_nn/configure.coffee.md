
## Configuration

Look at the file [DFSConfigKeys.java][keys] for an exhaustive list of supported
properties.

*   `site` (object)
    Properties added to the "hdfs-site.xml" file.
*   `opts` (string)
    NameNode options.

Example:

```json
{
  "ryba": {
    "hdfs":
      "nn": {
        "java_opts": "-Xms1024m -Xmx1024m",
        "include": ["in.my.cluster"],
        "exclude": "not.in.my.cluster"
    }
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

      # Layout
      options.pid_dir ?= service.deps.hadoop_core.options.hdfs.pid_dir
      options.log_dir ?= service.deps.hadoop_core.options.hdfs.log_dir
      options.conf_dir ?= '/etc/hadoop/conf'
      options.hadoop_conf_dir ?= '/etc/hadoop/conf'
      # Java
      options.java_home ?= service.deps.java.options.java_home
      options.hadoop_opts ?= service.deps.hadoop_core.options.hadoop_opts
      options.hadoop_namenode_init_heap ?= '-Xms1024m'
      options.heapsize ?= '1024m'
      options.newsize ?= '200m'
      options.java_opts ?= ""
      options.hadoop_heap ?= service.deps.hadoop_core.options.hadoop_heap
      # Misc
      options.clean_logs ?= false
      options.fqdn ?= service.node.fqdn
      options.hostname ?= service.node.hostname
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.hadoop_policy ?= {}
      options.hdfs_krb5_user = service.deps.hdfs[0].options.hdfs.krb5_user
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user

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

      # Hadoop core-site.xml
      options.core_site = merge {}, service.deps.hadoop_core.options.core_site, options.core_site or {}
      # Number of minutes after which the checkpoint gets deleted
      options.core_site['fs.trash.interval'] ?= '10080' #1 week
      # Hadoop hdfs-site.xml
      options.hdfs_site ?= {}
      options.hdfs_site['dfs.http.policy'] ?= 'HTTPS_ONLY' # HTTP_ONLY or HTTPS_ONLY or HTTP_AND_HTTPS
      # Data
      # Comma separated list of paths. Use the list of directories.
      # For example, /data/1/hdfs/nn,/data/2/hdfs/nn.
      options.hdfs_site['dfs.namenode.name.dir'] ?= ['/var/hdfs/name']
      options.hdfs_site['dfs.namenode.name.dir'] = options.hdfs_site['dfs.namenode.name.dir'].join ',' if Array.isArray options.hdfs_site['dfs.namenode.name.dir']
      # Network
      options.slaves = service.deps.hdfs_dn.map (srv) -> srv.node.fqdn
      options.hdfs_site['dfs.hosts'] ?= "#{options.conf_dir}/dfs.include"
      options.include ?= service.deps.hdfs_dn.map (srv) -> srv.node.fqdn
      options.include = string.lines options.include if typeof options.include is 'string'
      options.hdfs_site['dfs.hosts.exclude'] ?= "#{options.conf_dir}/dfs.exclude"
      options.exclude ?= []
      options.exclude = string.lines options.exclude if typeof options.exclude is 'string'
      options.hdfs_site['fs.permissions.umask-mode'] ?= '026' # 0750
      # If "true", access tokens are used as capabilities
      # for accessing datanodes. If "false", no access tokens are checked on
      # accessing datanodes.
      options.hdfs_site['dfs.block.access.token.enable'] ?= if options.core_site['hadoop.security.authentication'] is 'kerberos' then 'true' else 'false'
      options.hdfs_site['dfs.block.local-path-access.user'] ?= ''
      options.hdfs_site['dfs.namenode.safemode.threshold-pct'] ?= '0.99'
      # Fix HDP Companion File bug
      options.hdfs_site['dfs.https.namenode.https-address'] = null
      # Activate ACLs
      options.hdfs_site['dfs.namenode.acls.enabled'] ?= 'true'
      # was before in hdfs template
      options.hdfs_site['dfs.webhdfs.enabled'] ?= 'true'
      options.hdfs_site['dfs.client.read.shortcircuit.streams.cache.size'] ?= '4096'
      options.hdfs_site['dfs.blockreport.initialDelay'] ?= '120'
      options.hdfs_site['dfs.blocksize'] ?= '134217728'
      options.hdfs_site['dfs.dfs.bytes-per-checksum'] ?= '512'
      # options.hdfs_site['dfs.cluster.administrators'] ?= "#{options.user.name}"
      options.hdfs_site['dfs.replication.max'] ?= "50"
      options.hdfs_site['dfs.support.append'] ?= "true"
      options.hdfs_site['dfs.permissions.superusergroup'] ?= "#{options.user.name}"
      options.hdfs_site['dfs.permissions.enabled'] ?= "true"
      options.hdfs_site['dfs.namenode.write.stale.datanode.ratio'] ?= "1.0f"
      options.hdfs_site['dfs.namenode.stale.datanode.interval'] ?= "30000"
      options.hdfs_site['dfs.namenode.avoid.write.stale.datanode'] ?= "true"
      options.hdfs_site['dfs.namenode.avoid.read.stale.datanode'] ?= "true"
      options.hdfs_site['dfs.namenode.name.dir.restore'] ?= "true"
      options.hdfs_site['dfs.namenode.handler.count'] ?= "40"
      options.hdfs_site['dfs.namenode.checkpoint.size'] ?= "0.5"
      options.hdfs_site['dfs.namenode.checkpoint.txns'] ?= "1000000"
      options.hdfs_site['dfs.namenode.checkpoint.period'] ?= "21600"
      options.hdfs_site['dfs.heartbeat.interval'] ?= "3"
      options.hdfs_site['dfs.datanode.max.transfer.threads'] ?= "1024"

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      # Configuration in "hdfs-site.xml"
      options.hdfs_site['dfs.namenode.kerberos.principal'] ?= "nn/_HOST@#{options.krb5.realm}"
      options.hdfs_site['dfs.namenode.keytab.file'] ?= '/etc/security/keytabs/nn.service.keytab'
      options.hdfs_site['dfs.namenode.kerberos.internal.spnego.principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"
      options.hdfs_site['dfs.namenode.kerberos.https.principal'] = "HTTP/_HOST@#{options.krb5.realm}"
      options.hdfs_site['dfs.web.authentication.kerberos.principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"
      options.hdfs_site['dfs.web.authentication.kerberos.keytab'] ?= '/etc/security/keytabs/spnego.service.keytab'
      # options.opts.java_properties['java.security.auth.login.config'] ?= "#{options.conf_dir}/hdfs_nn_jaas.conf"

## Configuration for HDFS High Availability (HA)

Add High Availability specific properties to the "hdfs-site.xml" file. The
inserted properties are similar than the ones for a client or slave
configuration with the additionnal "dfs.namenode.shared.edits.dir" property.

The default configuration implement the "sshfence" fencing method. This method
SSHes to the target node and uses fuser to kill the process listening on the
service's TCP port.

      # HDFS Single Node configuration
      if service.instances.length is 1
        options.core_site['fs.defaultFS'] ?= "hdfs://#{service.node.fqdn}:8020"
        options.hdfs_site['dfs.ha.automatic-failover.enabled'] ?= 'false'
        options.hdfs_site['dfs.namenode.http-address'] ?= '0.0.0.0:50070'
        options.hdfs_site['dfs.namenode.https-address'] ?= '0.0.0.0:50470'
        options.hdfs_site['dfs.nameservices'] = null
      # HDFS HA configuration
      else if service.instances.length is 2
        throw Error "Required Option: options.nameservice" unless options.nameservice
        options.hdfs_site['dfs.nameservices'] ?= ''
        options.hdfs_site['dfs.nameservices'] += "#{options.nameservice} " unless options.nameservice in options.hdfs_site['dfs.nameservices'].split ' '
        options.hdfs_site['dfs.nameservices'] = options.hdfs_site['dfs.nameservices'].trim()
        options.core_site['fs.defaultFS'] ?= "hdfs://#{options.nameservice}"
        options.active_nn_host ?= service.instances[0].node.fqdn
        options.standby_nn_host = service.instances.filter( (instance) -> instance.node.fqdn isnt options.active_nn_host )[0].node.fqdn
        for srv in service.deps.hdfs_nn
          srv.options.hostname ?= srv.node.hostname
        # for srv in service.deps.hdfs_jn
        #   options.hdfs_site['dfs.journalnode.kerberos.principal'] ?= srv.options.hdfs_site['dfs.journalnode.kerberos.principal']
      else throw Error "Invalid number of NanodeNodes, got #{service.instances.length}, expecting 2"

Since [HDFS-6376](https://issues.apache.org/jira/browse/HDFS-6376),
Nameservice must be explicitely set as internal to provide other nameservices,
for distcp purpose.

      options.hdfs_site['dfs.internal.nameservices'] ?= ''
      if options.nameservice not in options.hdfs_site['dfs.internal.nameservices'].split ','
        options.hdfs_site['dfs.internal.nameservices'] += "#{if options.hdfs_site['dfs.internal.nameservices'] isnt '' then ',' else ''}#{options.nameservice}"
      options.hdfs_site["dfs.ha.namenodes.#{options.nameservice}"] = (for srv in service.deps.hdfs_nn then srv.options.hostname).join ','
      for srv in service.deps.hdfs_nn
        options.hdfs_site['dfs.namenode.http-address'] = null
        options.hdfs_site['dfs.namenode.https-address'] = null
        options.hdfs_site["dfs.namenode.rpc-address.#{options.nameservice}.#{srv.options.hostname}"] ?= "#{srv.node.fqdn}:8020"
        options.hdfs_site["dfs.namenode.http-address.#{options.nameservice}.#{srv.options.hostname}"] ?= "#{srv.node.fqdn}:50070"
        options.hdfs_site["dfs.namenode.https-address.#{options.nameservice}.#{srv.options.hostname}"] ?= "#{srv.node.fqdn}:50470"
        options.hdfs_site["dfs.client.failover.proxy.provider.#{options.nameservice}"] ?= 'org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider'
      options.hdfs_site['dfs.ha.automatic-failover.enabled'] ?= 'true'
      options.hdfs_site['dfs.namenode.shared.edits.dir'] = (for srv in service.deps.hdfs_jn then "#{srv.node.fqdn}:#{srv.options.hdfs_site['dfs.journalnode.rpc-address'].split(':')[1]}").join ';'
      options.hdfs_site['dfs.namenode.shared.edits.dir'] = "qjournal://#{options.hdfs_site['dfs.namenode.shared.edits.dir']}/#{options.nameservice}"

## SSL

      options.ssl = merge {}, service.deps.hadoop_core.options.ssl, options.ssl
      options.ssl.conf_dir ?= '/etc/security/serverKeys'
      options.ssl_server = merge {}, service.deps.hadoop_core.options.ssl_server, options.ssl_server or {},
        'ssl.server.keystore.location': "#{options.ssl.conf_dir}/hdfs-namenode-keystore"
        'ssl.server.truststore.location': "#{options.ssl.conf_dir}/hdfs-namenode-truststore"
      options.ssl_client = merge {}, service.deps.hadoop_core.options.ssl_client, options.ssl_client or {},
        'ssl.client.truststore.location': "#{options.ssl.conf_dir}/hdfs-namenode-truststore"

## Export configuration

      for srv in service.deps.hdfs_dn
        for property in [
          'dfs.namenode.kerberos.principal'
          'dfs.namenode.kerberos.internal.spnego.principal'
          'dfs.namenode.kerberos.https.principal'
          'dfs.web.authentication.kerberos.principal'
          'dfs.ha.automatic-failover.enabled'
          'dfs.nameservices'
          'dfs.internal.nameservices'
          'fs.permissions.umask-mode'
          'dfs.block.access.token.enable'
        ] then srv.options.hdfs_site[property] ?= options.hdfs_site[property]
        for property in [
          'fs.defaultFS'
        ] then srv.options.core_site[property] ?= options.core_site[property]
        for property of options.hdfs_site
          ok = false
          ok = true if /^dfs\.namenode\.\w+-address/.test property
          ok = true if property.indexOf('dfs.ha.namenodes.') is 0
          continue unless ok
          srv.options.hdfs_site[property] = options.hdfs_site[property]

      for srv in service.deps.hdfs_jn
        for property in [
          'dfs.namenode.kerberos.principal'
          'dfs.nameservices'
          'dfs.internal.nameservices'
          'fs.permissions.umask-mode'
          'dfs.block.access.token.enable'
        ] then srv.options.hdfs_site[property] ?= options.hdfs_site[property]
        for property in [
          'fs.defaultFS'
        ] then srv.options.core_site[property] ?= options.core_site[property]
        for property of options.hdfs_site
          ok = false
          ok = true if /^dfs\.namenode\.\w+-address/.test property
          ok = true if property.indexOf('dfs.ha.namenodes.') is 0
          continue unless ok
          srv.options.hdfs_site[property] = options.hdfs_site[property]

## Test

      options.test = merge {}, service.deps.test_user.options, options.test or {}

## Wait

      options.wait_zookeeper_server = service.deps.zookeeper_server[0].options.wait
      options.wait_hdfs_jn = service.deps.hdfs_jn[0].options.wait
      options.wait_hdfs_dn = service.deps.hdfs_dn[0].options.wait
      options.wait = {}
      options.wait.conf_dir = options.conf_dir
      options.wait.ipc = for srv in service.deps.hdfs_nn
        nameservice =  if options.nameservice then ".#{options.nameservice}" or ''
        hostname = if options.nameservice then ".#{srv.node.hostname}" else ''
        srv.options.hdfs_site ?= {}
        if srv.options.hdfs_site["dfs.namenode.rpc-address#{nameservice}#{hostname}"]
          [fqdn, port] = srv.options.hdfs_site["dfs.namenode.rpc-address#{nameservice}#{hostname}"].split(':')
        else
          fqdn = srv.node.fqdn
          port = 8020
        host: fqdn, port: port
      options.wait.http = for srv in service.deps.hdfs_nn
        protocol = if options.hdfs_site['dfs.http.policy'] is 'HTTP_ONLY' then 'http' else 'https'
        nameservice =  if options.nameservice then ".#{options.nameservice}" or ''
        hostname = if options.nameservice then ".#{srv.node.hostname}" else ''
        srv.options.hdfs_site ?= {}
        if srv.options.hdfs_site["dfs.namenode.rpc-address#{nameservice}#{hostname}"]
          [fqdn, port] = srv.options.hdfs_site["dfs.namenode.#{protocol}-address#{nameservice}#{hostname}"].split(':')
        else
          fqdn = srv.node.fqdn
          port = if options.hdfs_site['dfs.http.policy'] is 'HTTP_ONLY' then '50070' else '50470'
        host: fqdn, port: port
      options.wait.krb5_user = service.deps.hadoop_core.options.hdfs.krb5_user

## Ambari Configuration

      options.configurations ?= {}
      # Env
      options.configurations['hadoop-env'] ?= {}
      # options.configurations['hadoop-env']['HADOOP_SECURE_DN_USER'] ?= options.user.name
      # options.configurations['hadoop-env']['HADOOP_SECURE_DN_LOG_DIR'] ?= options.log_dir
      options.configurations['hadoop-env']['java_home'] ?= options.java_home
      # options.configurations['hadoop-env']['HADOOP_NAMENODE_INIT_HEAPSIZE'] ?= options.hadoop_namenode_init_heap
      options.configurations['hadoop-env']['namenode_newsize'] ?= options.newsize
      # Ambari required
      options.configurations['hadoop-env']['namenode_heapsize'] ?= options.heapsize
      options.configurations['hadoop-env']['namenode_opt_newsize'] ?= options.newsize
      options.configurations['hadoop-env']['namenode_opt_maxnewsize'] ?= options.newsize
      #HA
      options.configurations['hadoop-env']['dfs_ha_initial_namenode_active'] ?= service.instances.filter( (instance) -> instance.node.fqdn is options.active_nn_host )[0].node.fqdn
      options.configurations['hadoop-env']['dfs_ha_initial_namenode_standby'] ?= service.instances.filter( (instance) -> instance.node.fqdn isnt options.active_nn_host )[0].node.fqdn

## Dependencies

    string = require 'nikita/lib/misc/string'
    {merge} = require 'nikita/lib/misc'
    appender = require 'ryba/lib/appender'
