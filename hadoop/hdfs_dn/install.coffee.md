
# Hadoop HDFS DataNode Install

A DataNode manages the storage attached to the node it run on. There
are usually one DataNode per node in the cluster. HDFS exposes a file
system namespace and allows user data to be stored in files. Internally,
a file is split into one or more blocks and these blocks are stored in
a set of DataNodes. The DataNodes also perform block creation, deletion,
and replication upon instruction from the NameNode.

In a Hight Availabity (HA) enrironment, in order to provide a fast
failover, it is necessary that the Standby node have up-to-date
information regarding the location of blocks in the cluster. In order
to achieve this, the DataNodes are configured with the location of both
NameNodes, and send block location information and heartbeats to both.

    module.exports = header: 'HDFS DN Ambari Install', handler: (options) ->

## Register

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## IPTables

| Service   | Port       | Proto     | Parameter                  |
|-----------|------------|-----------|----------------------------|
| datanode  | 50010/1004 | tcp/http  | dfs.datanode.address       |
| datanode  | 50075/1006 | tcp/http  | dfs.datanode.http.address  |
| datanode  | 50475      | tcp/https | dfs.datanode.https.address |
| datanode  | 50020      | tcp       | dfs.datanode.ipc.address   |

The "dfs.datanode.address" default to "50010" in non-secured mode. In non-secured
mode, it must be set to a value below "1024" and default to "1004".

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      [_, dn_address] = options.hdfs_site['dfs.datanode.address'].split ':'
      [_, dn_http_address] = options.hdfs_site['dfs.datanode.http.address'].split ':'
      [_, dn_https_address] = options.hdfs_site['dfs.datanode.https.address'].split ':'
      [_, dn_ipc_address] = options.hdfs_site['dfs.datanode.ipc.address'].split ':'
      @tools.iptables
        header: 'IPTables'
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: dn_address, protocol: 'tcp', state: 'NEW', comment: "HDFS DN Data" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: dn_http_address, protocol: 'tcp', state: 'NEW', comment: "HDFS DN HTTP" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: dn_https_address, protocol: 'tcp', state: 'NEW', comment: "HDFS DN HTTPS" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: dn_ipc_address, protocol: 'tcp', state: 'NEW', comment: "HDFS DN Meta" }
        ]
        if: options.iptables

## Packages

Install the "hadoop-hdfs-datanode" service, symlink the rc.d startup script
inside "/etc/init.d" and activate it on startup.

      @call header: 'Packages', ->
        @service
          name: 'hadoop-hdfs-datanode'
        @hdp_select
          name: 'hadoop-hdfs-client' # Not checked
          name: 'hadoop-hdfs-datanode'

      @call header: 'Compression', retry: 2, ->
        @service.remove 'snappy', if: options.attempt is 1
        @service name: 'snappy'
        @service name: 'snappy-devel'
        @system.link
          source: '/usr/lib64/libsnappy.so'
          target: '/usr/hdp/current/hadoop-client/lib/native/.'
        @call (_, callback) ->
          @service
            name: 'lzo-devel'
            relax: true
          , (err) ->
            @service.remove
              if: !!err
              name: 'lzo-devel'
            @next callback
        @service
          name: 'hadooplzo'
        @service
          name: 'hadooplzo-native'

## Layout

Create the DataNode data and pid directories. The data directory is set by the
"hdp.hdfs.site['dfs.datanode.data.dir']" and default to "/var/hdfs/data". The
pid directory is set by the "hdfs\_pid\_dir" and default to "/var/run/hadoop-hdfs"

      @call header: 'Layout', ->
        # no need to restrict parent directory and yarn will complain if not accessible by everyone
        pid_dir = options.secure_dn_pid_dir
        pid_dir = pid_dir.replace '$USER', options.user.name
        pid_dir = pid_dir.replace '$HADOOP_SECURE_DN_USER', options.user.name
        pid_dir = pid_dir.replace '$HADOOP_IDENT_STRING', options.user.name
        # TODO, in HDP 2.1, datanode are started as root but in HDP 2.2, we should
        # start it as HDFS and use JAAS
        @system.mkdir
          target: for dir in options.hdfs_site['dfs.datanode.data.dir'].split ','
            if dir.indexOf('file://') is 0
              dir.substr(7) 
            else if dir.indexOf('file://') is -1
              dir
            else 
              dir.substr(dir.indexOf('file://')+7)
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0750
          parent: true
        @system.tmpfs
          if_os: name: ['redhat','centos'], version: '7'
          mount: pid_dir
          uid: options.user.name
          gid: options.hadoop_group.name
          perm: '0750'
        @system.mkdir
          target: "#{pid_dir}"
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o0755
          parent: true
        @system.mkdir
          target: "#{options.log_dir}" #/#{options.user.name}
          uid: options.user.name
          gid: options.group.name
          parent: true
        @system.mkdir
          target: "#{path.dirname options.hdfs_site['dfs.domain.socket.path']}"
          uid: options.user.name
          gid: options.hadoop_group.name
          mode: 0o751
          parent: true

## Kerberos

Create the DataNode service principal in the form of "dn/{host}@{realm}" and place its
keytab inside "/etc/security/keytabs/dn.service.keytab" with ownerships set to "hdfs:hadoop"
and permissions set to "0600".

        @krb5.addprinc
          header: 'Kerberos'
          principal: options.krb5.principal
          randkey: true
          keytab: options.krb5.keytab
          uid: options.user.name
          gid: options.group.name
          mode: 0o0600
        , options.krb5.admin

# Kernel

Configure kernel parameters at runtime. There are no properties set by default,
here's a suggestion:

*    vm.swappiness = 10
*    vm.overcommit_memory = 1
*    vm.overcommit_ratio = 100
*    net.core.somaxconn = 4096 (default socket listen queue size 128)

Note, we might move this middleware to Masson.

      @tools.sysctl
        header: 'Kernel'
        properties: options.sysctl
        merge: true
        comment: true

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

This is a dirty fix of [this bug][jsvc-192].
When launched with -user parameter, jsvc downgrades user via setuid() system call,
but the operating system limits (max number of open files, for example) remains the same.
As jsvc is used by bigtop scripts to run hdfs via root, we also (in fact: only) 
need to fix limits to root account, until Bigtop integrates jsvc 1.0.6

      @system.limits
        header: 'Ulimit to root'
        user: 'root'
      , options.user.limits

### DATANODE component wait
Wait for the DATANODE component to be declared on the host

      @ambari.hosts.component_wait
        header: 'DATANODE WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'DATANODE'
        hostname: options.fqdn

### JOURNALNODE component install
Put the JOURNALNODE component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'DATANODE set installed'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'DATANODE'
        hostname: options.fqdn

## Dependencies

    misc = require 'nikita/lib/misc'
    path = require 'path'

[key_os]: http://fr.slideshare.net/vgogate/hadoop-configuration-performance-tuning
[jsvc-192]: https://issues.apache.org/jira/browse/DAEMON-192
