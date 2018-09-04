
# HBase Master Install

TODO: [HBase backup node](http://willddy.github.io/2013/07/02/HBase-Add-Backup-Master-Node.html)

    module.exports =  header: 'HBase Master Install', handler: ({options}) ->

## Register

      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['file', 'jaas'], 'ryba/lib/file_jaas'

## IPTables

| Service             | Port  | Proto | Info                   |
|---------------------|-------|-------|------------------------|
| HBase Master        | 60000 | http  | hbase.master.port      |
| HMaster Info Web UI | 60010 | http  | hbase.master.info.port |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.hbase_site['hbase.master.port'], protocol: 'tcp', state: 'NEW', comment: "HBase Master" }
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.hbase_site['hbase.master.info.port'], protocol: 'tcp', state: 'NEW', comment: "HMaster Info Web UI" }
        ]
      

## HBase Master Layout

      @system.tmpfs
        header: 'Run dir'
        if_os: name: ['redhat','centos'], version: '7'
        mount: options.pid_dir
        uid: options.user.name
        gid: options.group.name
        perm: '0755'

# ## Zookeeper JAAS
#  for now let ambari create jaas
# JAAS configuration files for zookeeper to be deployed on the HBase Master,
# RegionServer, and HBase client host machines.
# 
# Environment file is enriched by "ryba-ambari-takeover/hbase" # HBase # Env".
# 
#       @file.jaas
#         header: 'Zookeeper JAAS'
#         target: "#{options.conf_dir}/hbase-master.jaas"
#         content: Client:
#           principal: options.hbase_site['hbase.master.kerberos.principal'].replace '_HOST', options.fqdn
#           keyTab: options.hbase_site['hbase.master.keytab.file']
#         uid: options.user.name
#         gid: options.group.name
#         mode: 0o600

## Kerberos

https://blogs.apache.org/hbase/entry/hbase_cell_security
https://hbase.apache.org/book/security.html

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos Master User'
        principal: options.hbase_site['hbase.master.kerberos.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.hbase_site['hbase.master.keytab.file']
        uid: options.user.name
        gid: options.hadoop_group.name

## SPNEGO

Ensure we have read access to the spnego keytab soring the server HTTP
principal.

      @system.execute
        header: 'SPNEGO'
        cmd: "su -l #{options.user.name} -c 'test -r /etc/security/keytabs/spnego.service.keytab'"

# User limits

      @system.limits
        header: 'Ulimit'
        user: options.user.name
      , options.user.limits

# Install Component

      @ambari.hosts.component_wait
        header: 'HBASE_MASTER WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HBASE_MASTER'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'HBASE_MASTER INSTALL'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HBASE_MASTER'
        hostname: options.fqdn

# Dependencies

    path = require 'path'
    quote = require 'regexp-quote'
