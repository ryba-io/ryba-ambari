# Hadoop HDFS JournalNode Install

It apply to a secured HDFS installation with Kerberos.

The JournalNode daemon is relatively lightweight, so these daemons may reasonably
be collocated on machines with other Hadoop daemons, for example NameNodes, the
JobTracker, or the YARN ResourceManager.

There must be at least 3 JournalNode daemons, since edit log modifications must
be written to a majority of JNs. To increase the number of failures a system
can tolerate, deploy an odd number of JNs because the system can tolerate at
most (N - 1) / 2 failures to continue to function normally.

    module.exports = header: 'HDFS JN Ambari Install', handler: ({options}) ->

## Register

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## IPTables

| Service     | Port | Proto  | Parameter                                      |
|-------------|------|--------|------------------------------------------------|
| journalnode | 8485 | tcp    | hdp.hdfs.site['dfs.journalnode.rpc-address']   |
| journalnode | 8480 | tcp    | hdp.hdfs.site['dfs.journalnode.http-address']  |
| journalnode | 8481 | tcp    | hdp.hdfs.site['dfs.journalnode.https-address'] |

Note, "dfs.journalnode.rpc-address" is used by "dfs.namenode.shared.edits.dir".

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      rpc = options.hdfs_site['dfs.journalnode.rpc-address'].split(':')[1]
      http = options.hdfs_site['dfs.journalnode.http-address'].split(':')[1]
      https = options.hdfs_site['dfs.journalnode.https-address'].split(':')[1]
      @tools.iptables
        header: 'IPTables'
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: rpc, protocol: 'tcp', state: 'NEW', comment: "HDFS JournalNode" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: http, protocol: 'tcp', state: 'NEW', comment: "HDFS JournalNode" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: https, protocol: 'tcp', state: 'NEW', comment: "HDFS JournalNode" }
        ]
        if: options.iptables

## Layout

The JournalNode data are stored inside the directory defined by the
"dfs.journalnode.edits.dir" property.

      @call header: 'Layout', ->
        @system.mkdir
          target: for dir in options.hdfs_site['dfs.journalnode.edits.dir'].split ','
            if dir.indexOf('file://') is 0
            then dir.substr(7) else dir
          uid: options.user.name
          gid: options.hadoop_group.name
        @system.mkdir
          target: "#{options.pid_dir}"
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0755
          parent: true
        @system.mkdir
          target: "#{options.log_dir}"
          uid: options.user.name
          gid: options.group.name
          parent: true

## Service

Install the "hadoop-hdfs-journalnode" service, symlink the rc.d startup script
inside "/etc/init.d" and activate it on startup.

      @call header: 'Packages', ->
        @service
          name: 'hadoop-hdfs-journalnode'
        @hdp_select
          name: 'hadoop-hdfs-client' # Not checked
          name: 'hadoop-hdfs-journalnode'
        @system.tmpfs
          if_os: name: ['redhat','centos'], version: '7'
          header: 'Run Dir'
          mount: "#{options.pid_dir}"
          uid: options.user.name
          gid: options.group.name
          perm: '0755'

## Kerberos

Create the JournalNode service principal in the form of "djn/{host}@{realm}" and place its
keytab inside "/etc/security/keytabs/jn.service.keytab" with ownerships set to "hdfs:hadoop"
and permissions set to "0600".

      @krb5.addprinc options.krb5.admin,
        header: 'Krb5 Service'
        principal: options.hdfs_site['dfs.journalnode.kerberos.principal'].replace '_HOST', options.fqdn
        keytab: options.hdfs_site['dfs.journalnode.keytab.file']
        randkey: true
        uid: options.user.name
        gid: options.hadoop_group.name
        mode: 0o0600



### JOURNALNODE component wait
Wait for the JOURNALNODE component declared on the host

      @ambari.hosts.component_wait
        header: 'JOURNALNODE WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'JOURNALNODE'
        hostname: options.fqdn

### JOURNALNODE component install
Put the JOURNALNODE component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'JOURNALNODE set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'JOURNALNODE'
        hostname: options.fqdn

