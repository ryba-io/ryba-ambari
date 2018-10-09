
# Ambari Logsearch Feeder Install

    module.exports =  header: 'Ambari Infra Solr Install', handler: ({options}) ->
    
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

## IPtables

      @tools.iptables
        header: 'Ambari Infra'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.configurations['infra-solr-env']['infra_solr_port'], protocol: 'tcp', state: 'NEW', comment: "Ambari Infra Logsearch" }
        ]

### Kerberos Principal

      @krb5.addprinc options.krb5.admin,
        header: 'Keberos Principal'
        principal: options.configurations['infra-solr-env']['infra_solr_kerberos_principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.configurations['infra-solr-env']['infra_solr_kerberos_keytab']
        uid: options.user.name
        gid: options.hadoop_group.name

## SSL

      @call header: 'SSL', retry: 0, ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['infra-solr-env']['infra_solr_truststore_location']
          storepass: options.configurations['infra-solr-env']['infra_solr_truststore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['infra-solr-env']['infra_solr_keystore_location']
          storepass: options.configurations['infra-solr-env']['infra_solr_keystore_password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['infra-solr-env']['infra_solr_keystore_password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.user.name
          gid: options.group.name
        @java.keystore_add
          keystore: options.configurations['infra-solr-env']['infra_solr_keystore_location']
          storepass: options.configurations['infra-solr-env']['infra_solr_keystore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name

### INFRA_SOLR component wait
Wait for the INFRA_SOLR component to be declared on the host

      @ambari.hosts.component_wait
        header: 'INFRA_SOLR WAITED'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'INFRA_SOLR'
        hostname: options.fqdn

### INFRA_SOLR component install
Put the INFRA_SOLR component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'INFRA_SOLR set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'INFRA_SOLR'
        hostname: options.fqdn
