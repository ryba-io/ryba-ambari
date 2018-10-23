
# Ambari Logsearch Server Install

    module.exports =  header: 'Ambari Smartsense Explorer Server Install', handler: ({options}) ->
    
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
          { chain: 'INPUT', jump: 'ACCEPT', dport: 9060, protocol: 'tcp', state: 'NEW', comment: "SMARTSENSE ACTIVITY EXPLORER ui" }
        ]


## SSL

      @call header: 'SSL', retry: 0, ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['activity-zeppelin-site']['zeppelin.ssl.truststore.path']
          storepass: options.configurations['activity-zeppelin-site']['zeppelin.ssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.path']
          storepass: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
        @java.keystore_add
          keystore: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.path']
          storepass: options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"


### Kerberos Principal

      # @krb5.addprinc options.krb5.admin,
      #   header: 'Kerberos Principal'
      #   principal: options.configurations['activity-zeppelin-site']['logsearch_external_solr_kerberos_principal'].replace '_HOST', options.fqdn
      #   randkey: true
      #   keytab: options.configurations['activity-zeppelin-site']['logsearch_external_solr_kerberos_keytab']
      #   uid: options.user.name
      #   gid: options.hadoop_group.name

