
# Ambari Zeppelin Install

    module.exports =  header: 'Ambari Zeppelin Install', handler: ({options}) ->
      
## Register

      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## Iptables
      
      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: parseInt(options.configurations['zeppelin-config']['zeppelin.server.port']), protocol: 'tcp', state: 'NEW', comment: "HTTP ZEPPELIN" },
          { chain: 'INPUT', jump: 'ACCEPT', dport: parseInt(options.configurations['zeppelin-config']['zeppelin.server.ssl.port']), protocol: 'tcp', state: 'NEW', comment: "HTTPS ZEPPELIN" }
        ]

## SSL

      @call header: 'SSL', retry: 0, ->
        # Client: import certificate to all hosts
        @java.keystore_add
          keystore: options.configurations['zeppelin-config']['zeppelin.ssl.truststore.path']
          storepass: options.configurations['zeppelin-config']['zeppelin.ssl.truststore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name
        # Server: import certificates, private and public keys to hosts with a server
        @java.keystore_add
          keystore: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.path']
          storepass: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.password']
          key: "#{options.ssl.key.source}"
          cert: "#{options.ssl.cert.source}"
          keypass: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.password']
          name: "#{options.ssl.key.name}"
          local: "#{options.ssl.key.local}"
          uid: options.user.name
          gid: options.group.name
        @java.keystore_add
          keystore: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.path']
          storepass: options.configurations['zeppelin-config']['zeppelin.ssl.keystore.password']
          caname: "hadoop_root_ca"
          cacert: "#{options.ssl.cacert.source}"
          local: "#{options.ssl.cacert.local}"
          uid: options.user.name
          gid: options.group.name
          
# ## Kerberos
# 
#         @krb5.addprinc
#           header: 'Kerberos'
#           principal: options.configurations['zeppelin-env']['zeppelin.server.kerberos.principal'].replace '_HOST', options.fqdn
#           randkey: true
#           keytab: options.configurations['zeppelin-env']['zeppelin.server.kerberos.keytab']
#           uid: options.user.name
#           gid: options.group.name
#           mode: 0o0600
#         , options.krb5.admin

## Install Zeppelin server

      @ambari.hosts.component_install
        header: 'ZEPPELIN_MASTER Host Add'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZEPPELIN_MASTER'
        hostname: options.fqdn
