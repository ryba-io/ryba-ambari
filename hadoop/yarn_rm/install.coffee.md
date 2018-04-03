
# Hadoop YARN ResourceManager Install

    module.exports = header: 'YARN RM Ambari Install', handler: (options) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register 'hdfs_mkdir', 'ryba/lib/hdfs_mkdir'
      @registry.register ['file', 'jaas'], 'ryba/lib/file_jaas'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

## Identities

By default, the "hadoop-yarn-resourcemanager" package create the following entries:

```bash
cat /etc/passwd | grep yarn
yarn:x:2403:2403:Hadoop YARN User:/var/lib/hadoop-yarn:/bin/bash
cat /etc/group | grep hadoop
hadoop:x:499:hdfs
```

      @system.group header: 'Hadoop Group', options.hadoop_group
      @system.group header: 'Group', options.group
      @system.user header: 'User', options.user

## Ulimit

Increase ulimit for the HDFS user. The HDP package create the following
files:

```bash
cat /etc/security/limits.d/yarn.conf
yarn   - nofile 32768
yarn   - nproc  65536
```

Note, a user must re-login for those changes to be taken into account.

      @system.limits
        header: 'Ulimit'
        user: options.user.name
      , options.user.limits

## IPTables

| Service         | Port  | Proto  | Parameter                                     |
|-----------------|-------|--------|-----------------------------------------------|
| resourcemanager | 8025  | tcp    | yarn.resourcemanager.resource-tracker.address | x
| resourcemanager | 8050  | tcp    | yarn.resourcemanager.address                  | x
| scheduler       | 8030  | tcp    | yarn.resourcemanager.scheduler.address        | x
| resourcemanager | 8088  | http   | yarn.resourcemanager.webapp.address           | x
| resourcemanager | 8090  | https  | yarn.resourcemanager.webapp.https.address     |
| resourcemanager | 8141  | tcp    | yarn.resourcemanager.admin.address            | x

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      id = if options.yarn_site['yarn.resourcemanager.ha.enabled'] is 'true' then ".#{options.hostname}" else ''
      rules = []
      # Application
      rpc_port = options.yarn_site["yarn.resourcemanager.address#{id}"].split(':')[1]
      rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: rpc_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Application Submissions" }
      # Scheduler
      s_port = options.yarn_site["yarn.resourcemanager.scheduler.address#{id}"].split(':')[1]
      rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: s_port, protocol: 'tcp', state: 'NEW', comment: "YARN Scheduler" }
      # RM Scheduler
      admin_port = options.yarn_site["yarn.resourcemanager.admin.address#{id}"].split(':')[1]
      rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: admin_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Scheduler" }
      # HTTP
      if options.yarn_site['yarn.http.policy'] in ['HTTP_ONLY', 'HTTP_AND_HTTPS']
        http_port = options.yarn_site["yarn.resourcemanager.webapp.address#{id}"].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: http_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Web UI" }
      # HTTPS
      if options.yarn_site['yarn.http.policy'] in ['HTTPS_ONLY', 'HTTP_AND_HTTPS']
        https_port = options.yarn_site["yarn.resourcemanager.webapp.https.address#{id}"].split(':')[1]
        rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: https_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Web UI" }
      # Resource Tracker
      rt_port = options.yarn_site["yarn.resourcemanager.resource-tracker.address#{id}"].split(':')[1]
      rules.push { chain: 'INPUT', jump: 'ACCEPT', dport: rt_port, protocol: 'tcp', state: 'NEW', comment: "YARN RM Application Submissions" }
      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: rules

## Service

Install the "hadoop-yarn-resourcemanager" service, symlink the rc.d startup script
inside "/etc/init.d" and activate it on startup.

      @call header: 'Service', ->
        @service
          name: 'hadoop-yarn-resourcemanager'
        @hdp_select
          name: 'hadoop-yarn-client' # Not checked
          name: 'hadoop-yarn-resourcemanager'
        @system.tmpfs
          header: 'Run dir'
          if_os: name: ['redhat','centos'], version: '7'
          mount: "#{options.pid_dir}"
          uid: options.user.name
          gid: options.hadoop_group.name
          perm: '0755'

      @call header: 'Layout', ->
        @system.mkdir
          target: "#{options.pid_dir}"
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o755
        @system.mkdir
          target: "#{options.log_dir}"
          uid: options.user.name
          gid: options.group.name
          parent: true
        @file.touch
          target: "#{options.yarn_site['yarn.resourcemanager.nodes.include-path']}"
        @file.touch
          target: "#{options.yarn_site['yarn.resourcemanager.nodes.exclude-path']}"

## SSL

      @call header: 'SSL', ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.ssl_client['ssl.client.truststore.location']
          storepass: options.ssl_client['ssl.client.truststore.password']
          caname: 'hadoop_root_ca'
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.ssl_server['ssl.server.keystore.location']
          storepass: options.ssl_server['ssl.server.keystore.password']
          key: options.ssl.key.source
          cert: options.ssl.cert.source
          keypass: options.ssl_server['ssl.server.keystore.keypassword']
          name: options.ssl.key.name
          local: options.ssl.key.local
        @java.keystore_add
          keystore: options.ssl_server['ssl.server.keystore.location']
          storepass: options.ssl_server['ssl.server.keystore.password']
          caname: 'hadoop_root_ca'
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local

## Kerberos

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos'
        principal: options.yarn_site['yarn.resourcemanager.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.yarn_site['yarn.resourcemanager.keytab']
        uid: options.user.name
        gid: options.hadoop_group.name

## Kerberos JAAS

The JAAS file is used by the ResourceManager to initiate a secure connection 
with Zookeeper.

      @file.jaas
        header: 'Kerberos JAAS'
        target: "#{options.hadoop_conf_dir}/yarn-rm.jaas"
        content: Client:
          principal: options.yarn_site['yarn.resourcemanager.principal'].replace '_HOST', options.fqdn
          keyTab: options.yarn_site['yarn.resourcemanager.keytab']
        uid: options.user.name
        gid: options.hadoop_group.name

## Ranger YARN Plugin Install

      # @call
      #   if: -> @contexts('ryba/ranger/admin').length > 0
      # , ->
      #   @call -> options.yarn_plugin_is_master = true
      #   @call 'ryba/ranger/plugins/yarn/install'

## Node Labels HDFS Layout

      @hdfs_mkdir
        if: options.yarn_site['yarn.node-labels.enabled'] is 'true'
        header: 'HDFS node-labels'
        target: options.yarn_site['yarn.node-labels.fs-store.root-dir']
        mode: 0o700
        user: options.user.name
        group: options.group.name
        unless_exec: mkcmd.hdfs options.hdfs_krb5_user, "hdfs --config #{options.conf_dir} dfs -test -d #{options.yarn_site['yarn.node-labels.fs-store.root-dir']}"

### RESOURCEMANAGER component wait
Wait for the RESOURCEMANAGER component to be declared on the host

      @ambari.hosts.component_wait
        header: 'Component WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RESOURCEMANAGER'
        hostname: options.fqdn

### RESOURCEMANAGER component install
Put the RESOURCEMANAGER component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'Set installed'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'RESOURCEMANAGER'
        hostname: options.fqdn

      #fix overriden property by ambari when kerberos is installed
      # ats.service.keytab become yarn.service.keytab
      @ambari.configs.update
        header: 'Fix hadoop-env'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hadoop-env'
        cluster_name: options.cluster_name
        properties:
          'hdfs_principal_name': options.hdfs_krb5_user.name

## Dependencies

    {merge} = require 'nikita/lib/misc'

## Todo: WebAppProxy.

It semms like it is run as part of rm by default and could also be started
separately on an edge node.

*   yarn.web-proxy.address    WebAppProxy                                   host:port for proxy to AM web apps. host:port if this is the same as yarn.resourcemanager.webapp.address or it is not defined then the ResourceManager will run the proxy otherwise a standalone proxy server will need to be launched.
*   yarn.web-proxy.keytab     /etc/security/keytabs/web-app.service.keytab  Kerberos keytab file for the WebAppProxy.
*   yarn.web-proxy.principal  wap/_HOST@REALM.TLD                           Kerberos principal name for the WebAppProxy.


[capacity]: http://hadoop.apache.org/docs/r2.5.0/hadoop-yarn/hadoop-yarn-site/CapacityScheduler.html

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
