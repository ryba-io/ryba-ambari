
# Hadoop Core Install

    module.exports = header: 'Hadoop Security Install', handler: (options) ->

## Keytab Directory

      @system.mkdir
        header: 'Keytabs'
        target: '/etc/security/keytabs'
        uid: 'root'
        gid: 'root' # was hadoop_group.name
        mode: 0o0755

## SPNEGO

Create the SPNEGO service principal in the form of "HTTP/{host}@{realm}" and place its
keytab inside "/etc/security/keytabs/spnego.service.keytab" with ownerships set to "hdfs:hadoop"
and permissions set to "0660". We had to give read/write permission to the group because the
same keytab file is for now shared between hdfs and yarn services.

      @call header: 'SPNEGO', ->
        @krb5.addprinc
          principal: options.spnego.principal
          randkey: true
          keytab: options.spnego.keytab
          uid: 'root'
          gid: options.hadoop_group.name
          mode: 0o660 # need rw access for hadoop and mapred users
        , options.krb5.admin

# ## SSL
# 
#       @call header: 'SSL', retry: 0, ->
#         @system.mkdir
#           target: options.ssl.conf_dir
#           gid: options.hadoop_group.name
#         # Client: import certificate to all hosts
#         @java.keystore_add
#           keystore: options.ssl_client['ssl.client.truststore.location']
#           storepass: options.ssl_client['ssl.client.truststore.password']
#           caname: "hadoop_root_ca"
#           cacert: options.ssl.cacert.source
#           local: options.ssl.cacert.local
#         # Server: import certificates, private and public keys to hosts with a server
#         @java.keystore_add
#           keystore: options.ssl_server['ssl.server.keystore.location']
#           storepass: options.ssl_server['ssl.server.keystore.password']
#           # caname: "hadoop_root_ca"
#           # cacert: "#{options.ssl.cacert}"
#           key: options.ssl.key.source
#           cert: options.ssl.cert.source
#           keypass: options.ssl_server['ssl.server.keystore.keypassword']
#           name: options.ssl.key.name
#           local: options.ssl.key.local
#         @java.keystore_add
#           keystore: options.ssl_server['ssl.server.keystore.location']
#           storepass: options.ssl_server['ssl.server.keystore.password']
#           caname: "hadoop_root_ca"
#           cacert: options.ssl.cacert.source
#           local: options.ssl.cacert.local
# 
# ## Rack info
# 
#       @ambari.hosts.rack
#         header: "Set rack"
#         if: options.rack_info
#         url: options.ambari_url
#         username: 'admin'
#         password: options.ambari_admin_password
#         cluster_name: options.cluster_name
#         hostname: options.fqdn
#         rack_info: options.rack_info
