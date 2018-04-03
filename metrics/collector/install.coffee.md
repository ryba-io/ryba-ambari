
# Ambari Metrics Collector Install

    module.exports =  header: 'Ambari Metrics Collector Install', handler: (options) ->
    
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

### Kerberos Principal

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos Zookeeper Client'
        principal: options.configurations['ams-hbase-security-site']['ams.zookeeper.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.configurations['ams-hbase-security-site']['ams.zookeeper.keytab']
        uid: options.user.name
        gid: options.group.name

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos Master User'
        principal: options.configurations['ams-hbase-security-site']['hbase.master.kerberos.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.configurations['ams-hbase-security-site']['hbase.master.keytab.file']
        uid: options.user.name
        gid: options.group.name
      
      @system.copy
        header: 'Kerberos RegionServer User'
        source: options.configurations['ams-hbase-security-site']['hbase.master.keytab.file']
        target: options.configurations['ams-hbase-security-site']['hbase.regionserver.keytab.file']

      @system.copy
        header: 'Kerberos Client User'
        source: options.configurations['ams-hbase-security-site']['hbase.master.keytab.file']
        target: options.configurations['ams-hbase-security-site']['hbase.myclient.keytab']

### METRICS_COLLECTOR component wait
Wait for the NODEMANAGER component to be declared on the host

      @ambari.hosts.component_wait
        header: 'METRICS_COLLECTOR WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_COLLECTOR'
        hostname: options.fqdn

### METRICS_COLLECTOR component install
Put the METRICS_COLLECTOR component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'METRICS_COLLECTOR set installed'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_COLLECTOR'
        hostname: options.fqdn
