
# Apache Spark Configure
Follow Hortonworks [documentation][spark2-ssl] to enable wire encryption.

    module.exports = (service) ->
      options = service.options

## Identities

      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'spark'
      options.group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'spark'
      options.user.system ?= true
      options.user.comment ?= 'Spark User'
      options.user.home ?= '/var/lib/spark'
      options.user.groups ?= 'hadoop'
      options.user.gid ?= options.group.name

## Kerberos

      # Kerberos HDFS Admin
      options.hdfs_krb5_user ?= service.deps.hadoop_core.options.hdfs.krb5_user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user
      options.hadoop_group ?= service.deps.hdfs[0].options.hadoop_group

## Environment

      # Layout
      options.conf_dir ?= '/etc/spark/conf'
      # Misc
      options.hostname = service.node.hostname
      options.fqdn ?= service.node.fqdn
      # options.hdfs_defaultfs = service.deps.hdfs_nn[0].options.core_site['fs.defaultFS']

## Configuration

      options.conf ?= {}
      options.configurations ?= {}
      options.configurations['spark2-defaults'] ?= {}
      options.configurations['spark2-defaults']['spark.authenticate'] ?= 'true'
## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      options.krb5.principal ?= "spark@#{options.krb5.realm}"
      options.krb5.keytab ?= '/etc/security/keytabs/spark.headless.keytab'
      # options.krb5.principal ?= "spark/#{service.node.fqdn}@#{options.krb5.realm}"
      # options.krb5.keytab ?= '/etc/security/keytabs/spark.service.keytab'
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      throw Error 'Required Options: "password"' unless options.krb5.password
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      #artifact id
      options.identities ?= {}
      options.identities['spark2'] ?= {}
      options.identities['spark2']['principal'] ?= {}
      options.identities['spark2']['principal']['configuration'] ?= 'spark2-defaults/spark.history.kerberos.principal'
      options.identities['spark2']['principal']['type'] ?= 'user'
      options.identities['spark2']['principal']['local_username'] ?= options.user.name
      options.identities['spark2']['principal']['value'] ?= options.krb5.principal#options.spark.krb5_user.principal
      options.identities['spark2']['name'] ?= 'spark2user'
      options.identities['spark2']['keytab'] ?= {}
      options.identities['spark2']['keytab']['owner'] ?= {}
      options.identities['spark2']['keytab']['owner']['access'] ?= 'r' 
      options.identities['spark2']['keytab']['owner']['name'] ?= options.user.name 
      options.identities['spark2']['keytab']['group'] ?= {}
      options.identities['spark2']['keytab']['group']['access'] ?= 'r'
      options.identities['spark2']['keytab']['group']['name'] ?= options.hadoop_group.name
      options.identities['spark2']['keytab']['file'] ?= options.krb5.keytab
      options.identities['spark2']['keytab']['configuration'] ?= 'spark2-defaults/spark.history.kerberos.keytab'

## SSL
Enable SSL as it is fully supprted starting from spark2

      options.ssl = merge {}, service.deps.ssl.options, options.ssl
      options.conf['spark.ssl.enabled'] ?= "false" # `!!service.deps.ssl`
      options.conf['spark.ssl.enabledAlgorithms'] ?= "TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA"
      options.conf['spark.ssl.keyPassword'] ?= service.deps.ssl.options.keystore.password
      options.conf['spark.ssl.keyStore'] ?= "#{options.conf_dir}/keystore"
      options.conf['spark.ssl.keyStorePassword'] ?= service.deps.ssl.options.keystore.password
      options.conf['spark.ssl.protocol'] ?= "SSLv3"
      options.conf['spark.ssl.trustStore'] ?= "#{options.conf_dir}/truststore"
      options.conf['spark.ssl.trustStorePassword'] ?= service.deps.ssl.options.truststore.password

## Ambari REST API

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.stack_name = service.deps.ambari_server.options.stack_name
      options.stack_version = service.deps.ambari_server.options.stack_version

## Ambari Agent
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['spark'] ?= options.user
        srv.options.groups ?= {}
        srv.options.groups['spark'] ?= options.group

## Dependencies

    {merge} = require 'nikita/lib/misc'

## Documentation

[spark2-ssl]:(https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.4/bk_spark-component-guide/content/config-spark2-encryption.html)
