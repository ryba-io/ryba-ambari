
# Ambari Logsearch Feeder Install

    module.exports =  header: 'Ambari Logsearch Feeder Install', handler: ({options}) ->
    
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
        header: 'Keberos Principal'
        principal: options.configurations['logfeeder-env']['logfeeder_external_solr_kerberos_principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.configurations['logfeeder-env']['logfeeder_external_solr_kerberos_keytab']
        uid: options.user.name
        gid: options.hadoop_group.name

## SSL

      @call header: 'SSL', retry: 0, ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['logfeeder-env']['logfeeder_truststore_location']
          storepass: options.configurations['logfeeder-env']['logfeeder_truststore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['logfeeder-env']['logfeeder_keystore_location']
          storepass: options.configurations['logfeeder-env']['logfeeder_keystore_password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['logfeeder-env']['logfeeder_keystore_password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.user.name
          gid: options.group.name
        @java.keystore_add
          keystore: options.configurations['logfeeder-env']['logfeeder_keystore_location']
          storepass: options.configurations['logfeeder-env']['logfeeder_keystore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name

### LOGSEARCH_LOGFEEDER component wait
Wait for the NODEMANAGER component to be declared on the host

      @ambari.hosts.component_wait
        header: 'LOGSEARCH_LOGFEEDER WAITED'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'LOGSEARCH_LOGFEEDER'
        hostname: options.fqdn

### LOGSEARCH_LOGFEEDER component install
Put the LOGSEARCH_LOGFEEDER component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'LOGSEARCH_LOGFEEDER set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'LOGSEARCH_LOGFEEDER'
        hostname: options.fqdn
