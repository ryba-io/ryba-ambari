
# Sparj Livy Server Configure

    module.exports = (service) ->
      options = service.options

## Identities

      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'livy'
      options.group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'livy'
      options.user.system ?= true
      options.user.comment ?= 'Livy User'
      options.user.home ?= '/var/lib/livy'
      options.user.groups ?= 'hadoop'
      options.user.gid ?= options.group.name
      options.fqdn ?= service.node.fqdn

## Kerberos

      # Kerberos HDFS Admin
      options.hdfs_krb5_user ?= service.deps.hadoop_core.options.hdfs.krb5_user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user

      options.ranger_admin ?= service.deps.ranger_admin.options.admin if service.deps.ranger_admin
      options.ranger_install = service.deps.ranger_hive[0].options.install if service.deps.ranger_hive
      options.test = merge {}, service.deps.test_user.options, options.test
      options.hadoop_group = merge {}, service.deps.spark_service[0].options.hadoop_group, options.hadoop_group

## Configuration

      options.configurations ?= {}
      options.configurations['livy-conf'] ?= {}

      options.port ?= '8998'
      
## SSL

      options.ssl = merge {}, service.deps.ssl.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      # options.truststore ?= {}
      throw Error "Required Option: ssl.cert" if  not options.ssl.cert
      throw Error "Required Option: ssl.key" if not options.ssl.key
      throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
      throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
      throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
      options.configurations['livy-conf']['livy.keystore'] ?= '/etc/security/serverKeys/spark-livy-keystore.jks'
      options.configurations['livy-conf']['livy.keystore.password'] ?= options.ssl.keystore.password
      options.configurations['livy-conf']['livy.key-password'] ?=  options.ssl.keystore.password
      
## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      options.configurations['livy-conf']['livy.server.auth.type'] ?= 'kerberos'
      options.configurations['livy-conf']['livy.server.auth.kerberos.keytab'] ?= '/etc/security/keytabs/spnego.service.keytab'
      options.configurations['livy-conf']['livy.server.auth.kerberos.principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"
      options.configurations['livy-conf']['livy.server.launch.kerberos.keytab'] ?= '/etc/security/keytabs/livy.service.keytab'
      options.configurations['livy-conf']['livy.server.launch.kerberos.principal'] ?= "#{options.user.name}/_HOST@#{options.krb5.realm}"

      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v

      for srv in service.deps.spark_service

        srv.options.configurations['livy-conf'] ?= {}
        enrich_config options.configurations['livy-conf'], srv.options.configurations['livy-conf']

## Ambari Agent
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['livy'] ?= options.user
        srv.options.groups ?= {}
        srv.options.groups['livy'] ?= options.group

## Dependencies

    {merge} = require 'nikita/lib/misc'
