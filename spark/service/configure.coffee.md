
# Apache Spark Configure

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

## SSL

For now on spark 1.3, [SSL is falling][secu] even after distributing keystore
and truststore on worker nodes as suggested in official documentation.
Maybe we shall share and deploy public keys instead of just the cacert
Disabling for now 

Note: 20160928, wdavidw, there was some issue where truststore and keystore
usage was messed up, the code in install is fixed but ssl is still disable because
I have no time to test it.

      # options.ssl = merge {}, service.deps.ssl.options, options.ssl
      # options.conf['spark.ssl.enabled'] ?= "false" # `!!service.deps.ssl`
      # options.conf['spark.ssl.enabledAlgorithms'] ?= "MD5"
      # options.conf['spark.ssl.keyPassword'] ?= service.deps.ssl.options.keystore.password
      # options.conf['spark.ssl.keyStore'] ?= "#{options.conf_dir}/keystore"
      # options.conf['spark.ssl.keyStorePassword'] ?= service.deps.ssl.options.keystore.password
      # options.conf['spark.ssl.protocol'] ?= "SSLv3"
      # options.conf['spark.ssl.trustStore'] ?= "#{options.conf_dir}/truststore"
      # options.conf['spark.ssl.trustStorePassword'] ?= service.deps.ssl.options.truststore.password
## Dependencies

    {merge} = require 'nikita/lib/misc'
