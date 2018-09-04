
# Hadoop HDFS NameNode Install

This implementation configure an HA HDFS cluster, using the [Quorum Journal Manager (QJM)](qjm)
feature  to share edit logs between the Active and Standby NameNodes. Hortonworks
provides [instructions to rollback a HA installation][rollback] that apply to Ambari.

Worth to investigate:

*   [RPC Congestion Control with FairCallQueue](https://issues.apache.org/jira/browse/HADOOP-9640)
*   [RPC fair share](https://issues.apache.org/jira/browse/HADOOP-10598)

[rollback]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.3/bk_Monitoring_Hadoop_Book/content/monitor-ha-undoing_2x.html

    module.exports = header: 'HDFS NN Install', handler: ({options}) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

## Wait

      # @call 'ryba/hadoop/hdfs_jn/wait', once: true, options.wait_hdfs_jn

## IPTables

| Service  | Port  | Proto | Parameter                  |
| -------- | ----- | ----- | -------------------------- |
| namenode | 50070 | tcp   | dfs.namdnode.http-address  |
| namenode | 50470 | tcp   | dfs.namenode.https-address |
| namenode | 8020  | tcp   | fs.defaultFS               |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      unless options.nameservice
        [_, port_rcp] = options.core_site['fs.defaultFS'].split ':'
        [_, port_rcp] = options.hdfs_site['dfs.namenode.http-address'].split ':'
        [_, port_rcp] = options.hdfs_site['dfs.namenode.https-address'].split ':'
      else
        [_, port_rcp] = options.hdfs_site["dfs.namenode.rpc-address.#{options.nameservice}.#{options.hostname}"].split ':'
        [_, port_http] = options.hdfs_site["dfs.namenode.http-address.#{options.nameservice}.#{options.hostname}"].split ':'
        [_, port_https] = options.hdfs_site["dfs.namenode.https-address.#{options.nameservice}.#{options.hostname}"].split ':'
      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: port_rcp, protocol: 'tcp', state: 'NEW', comment: "HDFS NN IPC" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: port_http, protocol: 'tcp', state: 'NEW', comment: "HDFS NN HTTP" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: port_https, protocol: 'tcp', state: 'NEW', comment: "HDFS NN HTTPS" }
        ]

## Service

Install the "hadoop-hdfs-namenode" service, symlink the rc.d startup script
inside "/etc/init.d" and activate it on startup.

      @call header: 'Packages', ->
        @service
          name: 'hadoop-hdfs-namenode'
        @hdp_select
          name: 'hadoop-hdfs-client' # Not checked
          name: 'hadoop-hdfs-namenode'
        @system.tmpfs
          if_os: name: ['redhat','centos'], version: '7'
          header: 'Run dir'
          mount: options.pid_dir
          uid: options.user.name
          gid: options.hadoop_group.name
          perm: '0750'

## Layout

Create the NameNode data and pid directories. The NameNode data is by defined in the
"/etc/hadoop/conf/hdfs-site.xml" file by the "dfs.namenode.name.dir" property. The pid
file is usually stored inside the "/var/run/hadoop-hdfs/hdfs" directory.

      @call header: 'Layout', ->
        @system.mkdir
          target: for dir in options.hdfs_site['dfs.namenode.name.dir'].split ','
            if dir.indexOf('file://') is 0
            then dir.substr(7) else dir
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o755
          parent: true
        @system.mkdir
          target: "#{options.pid_dir.replace '$USER', options.user.name}"
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o755
        @system.mkdir
          target: "#{options.log_dir}"
          uid: options.user.name
          gid: options.group.name
          parent: true


## SSL

      @call header: 'SSL', retry: 0, ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.ssl_client['ssl.client.truststore.location']
          storepass: options.ssl_client['ssl.client.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.ssl_server['ssl.server.keystore.location']
          storepass: options.ssl_server['ssl.server.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.ssl_server['ssl.server.keystore.keypassword']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.ssl_server['ssl.server.keystore.location']
          storepass: options.ssl_server['ssl.server.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"

## Kerberos

Create a service principal for this NameNode. The principal is named after
"nn/#{fqdn}@#{realm}".

      @krb5.addprinc options.krb5.admin,
        header: 'Krb5 Service'
        principal: options.hdfs_site['dfs.namenode.kerberos.principal'].replace '_HOST', options.fqdn
        keytab: options.hdfs_site['dfs.namenode.keytab.file']
        randkey: true
        uid: options.user.name
        gid: options.hadoop_group.name
        mode: 0o0600

## Ulimit

Increase ulimit for the HDFS user. The HDP package create the following
files:

```bash
cat /etc/security/limits.d/hdfs.conf
hdfs   - nofile 32768
hdfs   - nproc  65536
```

The procedure follows [Kate Ting's recommandations][kate]. This is a cause
of error if you receive the message: 'Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread'.

Also worth of interest are the [Pivotal recommandations][hawq] as well as the
[Greenplum recommandation from Nixus Technologies][greenplum], the
[MapR documentation][mapr] and [Hadoop Performance via Linux presentation][hpl].

Note, a user must re-login for those changes to be taken into account.

      @system.limits
        header: 'Ulimit'
        user: options.user.name
      , options.user.limits

## Slaves

The slaves file should contain the hostname of every machine in the cluster
which should start TaskTracker and DataNode daemons.

Helper scripts (described below) use this file in "/etc/hadoop/conf/slaves"
to run commands on many hosts at once. In order to use this functionality, ssh
trusts (via either passphraseless ssh or some other means, such as Kerberos)
must be established for the accounts used to run Hadoop.

      # @file
      #   header: 'Slaves'
      #   content: options.slaves.join '\n'
      #   target: "#{options.conf_dir}/slaves"
      #   eof: true

# ## Policy
# 
# By default the service-level authorization is disabled in hadoop, to enable that
# we need to set/configure the hadoop.security.authorization to true in
# ${HADOOP_CONF_DIR}/core-site.xml
# 
#       @hconfigure
#         header: 'Policy'
#         if: options.core_site['hadoop.security.authorization'] is 'true'
#         target: "#{options.conf_dir}/hadoop-policy.xml"
#         source: "#{__dirname}/../../../resources/core_hadoop/hadoop-policy.xml"
#         local: true
#         properties: options.hadoop_policy
#         backup: true

### NAMENODE component wait
Wait for the NAMENODE component to be declared on the host

      @ambari.hosts.component_wait
        header: 'NAMENODE WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'NAMENODE'
        hostname: options.fqdn

### NAMENODE component install
Put the JOURNALNODE component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'NAMENODE set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'NAMENODE'
        hostname: options.fqdn

## Format

Format the HDFS filesystem. This command is only run from the active NameNode and if
this NameNode isn't yet formated by detecting if the "current/VERSION" exists. The action
is only exected once all the JournalNodes are started. The NameNode is finally restarted
if the NameNode was formated.

      any_dfs_name_dir = options.hdfs_site['dfs.namenode.name.dir'].split(',')[0]
      any_dfs_name_dir = any_dfs_name_dir.substr(7) if any_dfs_name_dir.indexOf('file://') is 0
      # For non HA mode
      @system.execute
        header: 'Format Active'
        cmd: "su -l #{options.user.name} -c \"hdfs namenode -format\""
        unless: options.nameservice
        unless_exists: "#{any_dfs_name_dir}/current/VERSION"
      # For HA mode, on the leader namenode
      @system.execute
        header: 'Format Standby'
        cmd: "su -l #{options.user.name} -c \"hdfs  namenode -format -clusterId '#{options.nameservice}'\""
        if: options.nameservice and options.active_nn_host is options.fqdn
        unless_exists: "#{any_dfs_name_dir}/current/VERSION"

## HA Init Standby NameNodes

Copy over the contents of the active NameNode metadata directories to an other,
unformatted NameNode. The command "hdfs namenode -bootstrapStandby" used for the transfer
is only executed on the standby NameNode.

      @call
        header: 'HA Init Standby'
        if: -> options.nameservice
        unless: -> options.fqdn is options.active_nn_host
      , ->
        @connection.wait
          host: options.active_nn_host
          port: 8020
        @system.execute
          cmd: "su -l #{options.user.name} -c \"hdfs namenode -bootstrapStandby -nonInteractive\""
          code_skipped: 5
          unless_exists: "#{any_dfs_name_dir}/current/VERSION"


## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
    {merge} = require 'nikita/lib/misc'
