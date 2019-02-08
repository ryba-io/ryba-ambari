
# Hadoop YARN ResourceManager Install

    module.exports = header: 'YARN RM Ambari Install', handler: ({options}) ->

```

Note, a user must re-login for those changes to be taken into account.

      @system.limits
        header: 'Ulimit'
        user: options.user.name
      , options.user.limits

## IPTables

| Service         | Port  | Proto  | Parameter                                     |
|-----------------|-------|--------|-----------------------------------------------|
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

# ## SSL
# 
#       @call header: 'SSL', ->
#         # Client: import certificate to all hosts
#         @java.keystore_add
#           keystore: options.ssl_client['ssl.client.truststore.location']
#           storepass: options.ssl_client['ssl.client.truststore.password']
#           caname: 'hadoop_root_ca'
#           cacert: options.ssl.cacert.source
#           local: options.ssl.cacert.local
#         # Server: import certificates, private and public keys to hosts with a server
#         @java.keystore_add
#           keystore: options.ssl_server['ssl.server.keystore.location']
#           storepass: options.ssl_server['ssl.server.keystore.password']
#           key: options.ssl.key.source
#           cert: options.ssl.cert.source
#           keypass: options.ssl_server['ssl.server.keystore.keypassword']
#           name: options.ssl.key.name
#           local: options.ssl.key.local
#         @java.keystore_add
#           keystore: options.ssl_server['ssl.server.keystore.location']
#           storepass: options.ssl_server['ssl.server.keystore.password']
#           caname: 'hadoop_root_ca'
#           cacert: options.ssl.cacert.source
#           local: options.ssl.cacert.local

# ## Kerberos
# 
#       @krb5.addprinc options.krb5.admin,
#         header: 'Kerberos'
#         unless: options.kerberos_managed
#         principal: options.yarn_site['yarn.resourcemanager.principal'].replace '_HOST', options.fqdn
#         randkey: true
#         keytab: options.yarn_site['yarn.resourcemanager.keytab']
#         uid: options.user.name
#         gid: options.hadoop_group.name

## Node Labels HDFS Layout

      # @hdfs_mkdir
      #   if: options.yarn_site['yarn.node-labels.enabled'] is 'true'
      #   header: 'HDFS node-labels'
      #   target: options.yarn_site['yarn.node-labels.fs-store.root-dir']
      #   mode: 0o700
      #   user: options.user.name
      #   group: options.group.name
      #   unless_exec: mkcmd.hdfs options.hdfs_krb5_user, "hdfs --config #{options.conf_dir} dfs -test -d #{options.yarn_site['yarn.node-labels.fs-store.root-dir']}"

# ### RESOURCEMANAGER component wait
# Wait for the RESOURCEMANAGER component to be declared on the host
# 
#       @ambari.hosts.component_wait
#         header: 'Component WAITED'
#         url: options.ambari_url
#         username: 'admin'
#         password: options.ambari_admin_password
#         cluster_name: options.cluster_name
#         component_name: 'RESOURCEMANAGER'
#         hostname: options.fqdn

# ### RESOURCEMANAGER component install
# Put the RESOURCEMANAGER component declared on the host as `INSTALLED` desired state
#
#       @ambari.hosts.component_install
#         if: options.takeover
#         header: 'Set installed'
#         url: options.ambari_url
#         username: 'admin'
#         password: options.ambari_admin_password
#         cluster_name: options.cluster_name
#         component_name: 'RESOURCEMANAGER'
#         hostname: options.fqdn
#
#       #fix overriden property by ambari when kerberos is installed
#       # ats.service.keytab become yarn.service.keytab
#       @ambari.configs.update
#         header: 'Fix hadoop-env'
#         if: options.takeover
#         url: options.ambari_url
#         username: 'admin'
#         password: options.ambari_admin_password
#         config_type: 'hadoop-env'
#         cluster_name: options.cluster_name
#         properties:
#           'hdfs_principal_name': options.hdfs_krb5_user.name

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
