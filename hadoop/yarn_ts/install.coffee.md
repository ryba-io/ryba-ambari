
# Hadoop YARN Timeline Server Install

The Timeline Server is a stand-alone server daemon and doesn't need to be
co-located with any other service.

    module.exports = header: 'YARN ATS Ambari Install', handler: ({options}) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'

## Wait

      @call once: true, 'masson/core/krb5_client/wait', options.wait_krb5_client

## IPTables

| Service   | Port   | Proto     | Parameter                                  |
|-----------|------- |-----------|--------------------------------------------|
| timeline  | 10200  | tcp/http  | yarn.timeline-service.address              |
| timeline  | 8188   | tcp/http  | yarn.timeline-service.webapp.address       |
| timeline  | 8190   | tcp/https | yarn.timeline-service.webapp.https.address |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      [_, rpc_port] = options.yarn_site['yarn.timeline-service.address'].split ':'
      [_, http_port] = options.yarn_site['yarn.timeline-service.webapp.address'].split ':'
      [_, https_port] = options.yarn_site['yarn.timeline-service.webapp.https.address'].split ':'
      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: rpc_port, protocol: 'tcp', state: 'NEW', comment: "Yarn Timeserver RPC" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: http_port, protocol: 'tcp', state: 'NEW', comment: "Yarn Timeserver HTTP" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: https_port, protocol: 'tcp', state: 'NEW', comment: "Yarn Timeserver HTTPS" }
        ]

## Service

Install the "hadoop-yarn-timelineserver" service, symlink the rc.d startup script
in "/etc/init.d/hadoop-hdfs-datanode" and define its startup strategy.

      @call header: 'Service', ->
        @service
          name: 'hadoop-yarn-timelineserver'
        @hdp_select
          name: 'hadoop-yarn-client' # Not checked
          name: 'hadoop-yarn-timelineserver'
        @system.tmpfs
          header: 'Run dir'
          if_os: name: ['redhat','centos'], version: '7'
          mount: "#{options.pid_dir}"
          uid: options.user.name
          gid: options.hadoop_group.name
          perm: '0755'

# Layout

      @call header: 'Layout', ->
        @system.mkdir
          target: "#{options.conf_dir}"
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
        @system.mkdir
          target: options.yarn_site['yarn.timeline-service.leveldb-timeline-store.path']
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0750
          parent: true

# HDFS Layout

See:

*   [YarnConfiguration](https://github.com/apache/hadoop/blob/trunk/hadoop-yarn-project/hadoop-yarn/hadoop-yarn-api/src/main/java/org/apache/hadoop/yarn/conf/YarnConfiguration.java#L1425-L1426)
*   [FileSystemApplicationHistoryStore](https://github.com/apache/hadoop/blob/trunk/hadoop-yarn-project/hadoop-yarn/hadoop-yarn-server/hadoop-yarn-server-applicationhistoryservice/src/main/java/org/apache/hadoop/yarn/server/applicationhistoryservice/FileSystemApplicationHistoryStore.java)

Note, this is not documented anywhere and might not be considered as a best practice.

      @call header: 'HDFS layout', ->
        return unless options.yarn_site['yarn.timeline-service.generic-application-history.store-class'] is "org.apache.hadoop.yarn.server.applicationhistoryservice.FileSystemApplicationHistoryStore"
        dir = options.yarn_site['yarn.timeline-service.fs-history-store.uri']
        @wait.execute
          cmd: mkcmd.hdfs options.hdfs_krb5_user, "hdfs --config #{options.conf_dir} dfs -test -d #{path.dirname dir}"
        @system.execute
          cmd: mkcmd.hdfs options.hdfs_krb5_user, """
          hdfs --config #{options.conf_dir} dfs -mkdir -p #{dir}
          hdfs --config #{options.conf_dir} dfs -chown #{options.user.name} #{dir}
          hdfs --config #{options.conf_dir} dfs -chmod 1777 #{dir}
          """
          unless_exec: "[[ hdfs  --config #{options.conf_dir} dfs -d #{dir} ]]"

## SSL

      @call header: 'SSL', ->
        @hconfigure
          target: "#{options.conf_dir}/ssl-server.xml"
          properties: options.ssl_server
        @hconfigure
          target: "#{options.conf_dir}/ssl-client.xml"
          properties: options.ssl_client
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.ssl_client['ssl.client.truststore.location']
          storepass: options.ssl_client['ssl.client.truststore.password']
          caname: "hadoop_root_ca"
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
          caname: "hadoop_root_ca"
          cacert: options.ssl.cacert.source
          local: options.ssl.cacert.local

## Kerberos

Create the Kerberos service principal by default in the form of
"ats/{host}@{realm}" and place its keytab inside
"/etc/security/keytabs/ats.service.keytab" with ownerships set to
"mapred:hadoop" and permissions set to "0600".

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos'
        principal: options.yarn_site['yarn.timeline-service.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.yarn_site['yarn.timeline-service.keytab']
        uid: options.user.name
        gid: options.group.name
        mode: 0o0600
        
### APP_TIMELINE_SERVER component wait
Wait for the APP_TIMELINE_SERVER component to be declared on the host

      @ambari.hosts.component_wait
        header: 'APP_TIMELINE_SERVER WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'APP_TIMELINE_SERVER'
        hostname: options.fqdn

### APP_TIMELINE_SERVER component install
Put the APP_TIMELINE_SERVER component declared on the host as `INSTALLED` desired state

      #fix overriden property by ambari when kerberos is installed
      # ats.service.keytab become yarn.service.keytab
      @ambari.configs.update
        header: 'Fix yarn-site'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'yarn-site'
        cluster_name: options.cluster_name
        properties:
          'yarn.timeline-service.keytab': options.yarn_site['yarn.timeline-service.keytab']
          'yarn.timeline-service.principal': options.yarn_site['yarn.timeline-service.principal']

      @ambari.hosts.component_install
        header: 'APP_TIMELINE_SERVER set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'APP_TIMELINE_SERVER'
        hostname: options.fqdn

## Dependencies

    path = require 'path'
    mkcmd = require 'ryba/lib/mkcmd'
