
# Ambari Logsearch Server Install

    module.exports =  header: 'Ambari Logsearch Server Install', handler: ({options}) ->
    
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

## IPTables

      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: parseInt(options.configurations['logsearch-env']['logsearch_ui_port']), protocol: 'tcp', state: 'NEW', comment: "Ambari Logsearch ui" }
        ]

      @file
        header: 'History Collection SolrConfig'
        local: true
        source: "#{__dirname}/../resources/history-solrconfig-#{options.download}.xml.j2"
        target: "/etc/ambari-logsearch-portal/conf/solr_configsets/history/conf/solrconfig.xml"
        context: 
          logsearch_service_logs_max_retention: 7
        backup: true
      @file
        header: 'History Collection managed-schema'
        local: true
        source: "#{__dirname}/../resources/history-managed-schema-#{options.download}.xml.j2"
        target: "/etc/ambari-logsearch-portal/conf/solr_configsets/history/conf/managed-schema"
        backup: true

## SSL

      @call header: 'SSL', retry: 0, ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['logsearch-env']['logsearch_truststore_location']
          storepass: options.configurations['logsearch-env']['logsearch_truststore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['logsearch-env']['logsearch_keystore_location']
          storepass: options.configurations['logsearch-env']['logsearch_keystore_password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['logsearch-env']['logsearch_keystore_password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.user.name
          gid: options.group.name
        @java.keystore_add
          keystore: options.configurations['logsearch-env']['logsearch_keystore_location']
          storepass: options.configurations['logsearch-env']['logsearch_keystore_password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name


### Kerberos Principal

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos Principal'
        principal: options.configurations['logsearch-env']['logsearch_external_solr_kerberos_principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.configurations['logsearch-env']['logsearch_external_solr_kerberos_keytab']
        uid: options.user.name
        gid: options.hadoop_group.name

      @krb5.addprinc options.krb5.admin,
        header: 'Keberos Principal'
        principal: options.configurations['infra-solr-env']['infra_solr_kerberos_principal'].replace '_HOST', options.fqdn
        randkey: true
        mode: 0o640
        keytab: options.configurations['infra-solr-env']['infra_solr_kerberos_keytab']
        uid: options.user.name
        gid: options.hadoop_group.name

### LOGSEARCH_SERVER component wait
Wait for the NODEMANAGER component to be declared on the host

      @ambari.hosts.component_wait
        header: 'LOGSEARCH_SERVER WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'LOGSEARCH_SERVER'
        hostname: options.fqdn

### LOGSEARCH_SERVER component install
Put the LOGSEARCH_SERVER component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'LOGSEARCH_SERVER set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'LOGSEARCH_SERVER'
        hostname: options.fqdn
