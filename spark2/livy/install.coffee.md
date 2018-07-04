

# Ambari Logsearch Server Install

    module.exports =  header: 'Ambari Spark Livy Server Install', handler: (options) ->
      
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
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
          { chain: 'INPUT', jump: 'ACCEPT', dport: parseInt(options.port), protocol: 'tcp', state: 'NEW', comment: "SPARK LIVY" }
        ]

      @ambari.configs.update
        header: 'Upload livy2-conf'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'livy2-conf'
        cluster_name: options.cluster_name
        properties: options.configurations['livy2-conf']

### Kerberos Principal

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos Principal'
        principal: options.configurations['livy-conf']['livy.server.launch.kerberos.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.configurations['livy-conf']['livy.server.launch.kerberos.keytab']
        uid: options.user.name
        gid: options.hadoop_group.name

## SSL

      @call header: 'SSL', retry: 0, ->
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['livy-conf']['livy.keystore']
          storepass: options.configurations['livy-conf']['livy.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['livy-conf']['livy.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.user.name
          gid: options.group.name
        @java.keystore_add
          keystore: options.configurations['livy-conf']['livy.keystore']
          storepass: options.configurations['livy-conf']['livy.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name
