
# Apache Spark Configure

    module.exports = (service) ->
      options = service.options

## Identities

      options.user ?= service.deps.spark[0].options.user
      options.group ?= service.deps.spark[0].options.group

## Kerberos

      # Kerberos HDFS Admin
      options.hdfs_krb5_user ?= service.deps.hadoop_core.options.hdfs.krb5_user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user

## Environment

      # Layout
      options.client_dir ?= '/usr/hdp/current/spark-client'
      options.conf_dir ?= '/etc/spark/conf'
      # Misc
      options.hostname = service.node.hostname
      options.fqdn ?= service.node.fqdn
      # options.hdfs_defaultfs = service.deps.hdfs_nn[0].options.core_site['fs.defaultFS']

## Test

      options.ranger_admin ?= service.deps.ranger_admin.options.admin if service.deps.ranger_admin
      options.ranger_install = service.deps.ranger_hive[0].options.install if service.deps.ranger_hive
      options.test = merge {}, service.deps.test_user.options, options.test
      # Hive Server2
      if service.deps.hive_server2
        options.hive_server2 = for srv in service.deps.hive_server2
          fqdn: srv.options.fqdn
          hostname: srv.options.hostname
          hive_site: srv.options.hive_site

## Configuration

      options.conf ?= {}

## SSL

For now on spark 1.3, [SSL is falling][secu] even after distributing keystore
and truststore on worker nodes as suggested in official documentation.
Maybe we shall share and deploy public keys instead of just the cacert
Disabling for now 

Note: 20160928, wdavidw, there was some issue where truststore and keystore
usage was messed up, the code in install is fixed but ssl is still disable because
I have no time to test it.

      options.ssl = merge {}, service.deps.ssl.options, options.ssl
      options.conf['spark.ssl.enabled'] ?= "false" # `!!service.deps.ssl`
      options.conf['spark.ssl.enabledAlgorithms'] ?= "MD5"
      options.conf['spark.ssl.keyPassword'] ?= service.deps.ssl.options.keystore.password
      options.conf['spark.ssl.keyStore'] ?= "#{options.conf_dir}/keystore"
      options.conf['spark.ssl.keyStorePassword'] ?= service.deps.ssl.options.keystore.password
      options.conf['spark.ssl.protocol'] ?= "SSLv3"
      options.conf['spark.ssl.trustStore'] ?= "#{options.conf_dir}/truststore"
      options.conf['spark.ssl.trustStorePassword'] ?= service.deps.ssl.options.truststore.password

## Ambari Configurations

      for srv in service.deps.spark
        srv.options.client_hosts ?= []
        srv.options.client_hosts.push service.node.fqdn if srv.options.client_hosts.indexOf(service.node.fqdn) is -1

## Ambari Server REST API

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal

## Ambari Agent
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['spark'] ?= options.user
        srv.options.groups ?= {}
        srv.options.groups['spark'] ?= options.group

## Wait

      options.wait_yarn_rm = service.deps.yarn_rm[0].options.wait
      options.wait_ranger_admin = service.deps.ranger_admin.options.wait if service.deps.ranger_admin

## Dependencies

    {merge} = require 'nikita/lib/misc'
