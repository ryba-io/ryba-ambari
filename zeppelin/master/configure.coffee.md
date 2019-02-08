
# Zeppelin Notebook Configure

    module.exports = (service) ->
      options = service.options

## Identities

      options.user ?= service.deps.zeppelin_service[0].options.user
      options.group ?= service.deps.zeppelin_service[0].options.group
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.fqdn ?= service.node.fqdn

## Configuration

      options.configurations ?= {}
      options.configurations['zeppelin-env'] ?= {}
      options.configurations['zeppelin-config'] ?= {}

      throw Error 'Unspecified zeppelin-config zeppelin.ssl.key.manager.password' unless service.deps.zeppelin_service[0].options.configurations['zeppelin-config']?['zeppelin.ssl.key.manager.password']
      options.configurations['zeppelin-config']['zeppelin.ssl.key.manager.password'] ?= service.deps.zeppelin_service[0].options.configurations['zeppelin-config'] 
      options.configurations['zeppelin-config'] ?= {}
      options.configurations['zeppelin-config']['zeppelin.server.port'] ?= '9996'
      options.configurations['zeppelin-config']['zeppelin.server.ssl.port'] ?= '9996'
      options.configurations['zeppelin-config']['zeppelin.spark.jar.dir'] ?= '/apps/zeppelin'
      
      

## SSL

      options.ssl = merge {}, service.deps.ssl.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      if options.ssl.enabled
      # options.truststore ?= {}
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
        throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
        options.configurations['zeppelin-config']['zeppelin.ssl'] ?= 'true'
        options.configurations['zeppelin-config']['zeppelin.ssl.keystore.password'] ?=  options.ssl.keystore.password
        options.configurations['zeppelin-config']['zeppelin.ssl.keystore.path'] ?= '/etc/security/serverKeys/zeppelin-keystore'
        options.configurations['zeppelin-config']['zeppelin.ssl.keystore.type'] ?= 'JKS'
        options.configurations['zeppelin-config']['zeppelin.ssl.truststore.password'] ?= options.ssl.truststore.password
        options.configurations['zeppelin-config']['zeppelin.ssl.truststore.path'] ?= '/etc/security/serverKeys/zeppelin-truststore'
        options.configurations['zeppelin-config']['zeppelin.ssl.truststore.type'] ?= 'JKS'

## Spark Livy Server Truststore

      if service.deps.zeppelin_service[0].options.livy_ssl_enabled
        options.configurations['zeppelin-config']['zeppelin.livy.ssl.trustStore'] ?= '/etc/security/serverKeys/zeppelin-truststore'
        options.configurations['zeppelin-config']['zeppelin.livy.ssl.trustStorePassword'] ?= options.configurations['zeppelin-config']['zeppelin.ssl.truststore.password']

## Kerberos

      # Kerberos HDFS Admin
      options.hdfs_krb5_user ?= service.deps.hadoop_core.options.hdfs.krb5_user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user
      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      options.configurations['zeppelin-env'] ?= {}
      options.configurations['zeppelin-env']['zeppelin.executor.mem'] ?= '1024m'

## Dependencies

    {merge} = require 'nikita/lib/misc'
