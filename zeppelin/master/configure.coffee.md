
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
      options.configurations['zeppelin-env']['zeppelin.server.kerberos.principal'] ?= service.deps.zeppelin_service[0].options.identities['zeppelin_master']['principal']['value']
      options.configurations['zeppelin-env']['zeppelin.server.kerberos.keytab'] ?= service.deps.zeppelin_service[0].options.identities['zeppelin_master']['keytab']['file']

## Ambari Configuration

      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v

      for srv in service.deps.zeppelin_service
        srv.options.configurations['zeppelin-config'] ?= {}
        srv.options.configurations['zeppelin-env'] ?= {}
        enrich_config options.configurations['zeppelin-config'], srv.options.configurations['zeppelin-config']
        enrich_config options.configurations['zeppelin-env'], srv.options.configurations['zeppelin-env']


        srv.options.master_hosts ?= []
        srv.options.master_hosts.push service.node.fqdn if srv.options.master_hosts.indexOf(service.node.fqdn) is -1

## Ambari rest api

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.stack_name = service.deps.ambari_server.options.stack_name
      options.stack_version = service.deps.ambari_server.options.stack_version
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal

## Wait

      options.wait ?= {}
      options.wait.http = 
        host: options.fqdn
        port: if options.ssl.enabled then options.configurations['zeppelin-config']['zeppelin.server.ssl.port'] else options.configurations['zeppelin-config']['zeppelin.server.port']

## Dependencies

    {merge} = require 'nikita/lib/misc'
