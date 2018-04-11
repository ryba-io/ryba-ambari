
# HBase RegionServer Install

    module.exports = header: 'HBase RegionServer Install', handler: (options) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['file', 'jaas'], 'ryba/lib/file_jaas'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'


## IPTables

| Service                      | Port  | Proto | Info                         |
|------------------------------|-------|-------|------------------------------|
| HBase Region Server          | 60020 | http  | hbase.regionserver.port      |
| HMaster Region Server Web UI | 60030 | http  | hbase.regionserver.info.port |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.hbase_site['hbase.regionserver.port'], protocol: 'tcp', state: 'NEW', comment: "HBase RegionServer" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.hbase_site['hbase.regionserver.info.port'], protocol: 'tcp', state: 'NEW', comment: "HBase RegionServer Info Web UI" }
        ]

## HBase Regionserver Layout

      @call header: 'Layout', ->
        @system.mkdir
          target: options.pid_dir
          uid: options.user.name
          gid: options.group.name
          mode: 0o0755
        @system.mkdir
          target: options.log_dir
          uid: options.user.name
          gid: options.group.name
          mode: 0o0755
        @system.mkdir
          target: options.conf_dir
          uid: options.user.name
          gid: options.group.name
          mode: 0o0755
        @system.mkdir
          target: "#{options.log_dir}/local/jars"
          uid: options.user.name
          gid: options.group.name
          mode: 0o0755

## Service

Install the "hbase-regionserver" service, symlink the rc.d startup script
inside "/etc/init.d" and activate it on startup.

      @call header: 'Service', ->
        @service
          name: 'hbase-regionserver'
        @hdp_select
          name: 'hbase-client'
        @hdp_select
          name: 'hbase-regionserver'
        @system.tmpfs
          header: 'Run dir'
          if_os: name: ['redhat','centos'], version: '7'
          mount: options.pid_dir
          uid: options.user.name
          gid: options.group.name
          perm: '0755'
# 
# ## Zookeeper JAAS
# 
# JAAS configuration files for zookeeper to be deployed on the HBase Master,
# RegionServer, and HBase client host machines.
# 
#       @file.jaas
#         header: 'Zookeeper JAAS'
#         target: "#{options.conf_dir}/hbase-regionserver.jaas"
#         content: Client:
#           principal: options.hbase_site['hbase.regionserver.kerberos.principal'].replace '_HOST', options.fqdn
#           keyTab: options.hbase_site['hbase.regionserver.keytab.file']
#         uid: options.user.name
#         gid: options.group.name

## Kerberos

      @system.copy
        header: 'Copy Keytab'
        if: options.copy_master_keytab
        source: options.copy_master_keytab
        target: options.hbase_site['hbase.regionserver.keytab.file']
      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos'
        unless: options.copy_master_keytab
        principal: options.hbase_site['hbase.regionserver.kerberos.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.hbase_site['hbase.regionserver.keytab.file']
        uid: options.user.name
        gid: options.hadoop_group.name

## RegionServers

Upload the list of registered RegionServers.

      regionservers = for fqdn, active of options.regionservers
        continue unless active
        fqdn
      @file
        header: 'Registered RegionServers'
        target: "#{options.conf_dir}/regionservers"
        content: (
          for fqdn, active of options.regionservers
            continue unless active
            fqdn
        ).join '\n'
        uid: options.user.name
        gid: options.hadoop_group.name
        eof: true
        mode: 0o640

# User limits

      @system.limits
        header: 'Ulimit'
        user: options.user.name
      , options.user.limits


# Install Component

      @ambari.hosts.component_wait
        header: 'HBASE_REGIONSERVER WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HBASE_REGIONSERVER'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'HBASE_REGIONSERVER INSTALL'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HBASE_REGIONSERVER'
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
          'hdfs_principal_name': options.hdfs_krb5_user.principal

# Module dependencies

    quote = require 'regexp-quote'
