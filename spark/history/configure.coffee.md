
# Apache Spark JOB History Server Configure

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

## Configuration

      options.conf ?= {}

## HDFS Log Dir
HDFS Directory from which the spark history server will load jobs information once in state finished.

      options.conf['spark.eventLog.dir'] ?= "#{service.deps.hdfs_nn[0].options.core_site['fs.defaultFS']}/user/#{options.user.name}/applicationHistory"
      options.conf['spark.history.fs.logDirectory'] ?= options.conf['spark.eventLog.dir']
## SSL

For now on spark 1.3, [SSL is falling][secu] even after distributing keystore
and truststore on worker nodes as suggested in official documentation.
Maybe we shall share and deploy public keys instead of just the cacert
Disabling for now 

Note: 20160928, wdavidw, there was some issue where truststore and keystore
usage was messed up, the code in install is fixed but ssl is still disable because
I have no time to test it.

      options.ssl = merge {}, service.deps.ssl.options, options.ssl
      # options.conf['spark.ssl.enabled'] ?= "false" # `!!service.deps.ssl`
      # options.conf['spark.ssl.enabledAlgorithms'] ?= "MD5"
      # options.conf['spark.ssl.keyPassword'] ?= service.deps.ssl.options.keystore.password
      # options.conf['spark.ssl.keyStore'] ?= "#{options.conf_dir}/keystore"
      # options.conf['spark.ssl.keyStorePassword'] ?= service.deps.ssl.options.keystore.password
      # options.conf['spark.ssl.protocol'] ?= "SSLv3"
      # options.conf['spark.ssl.trustStore'] ?= "#{options.conf_dir}/truststore"
      # options.conf['spark.ssl.trustStorePassword'] ?= service.deps.ssl.options.truststore.password

## Port

      options.conf['spark.history.ui.port'] ?= '18080'

## Kerberos
Note: lucasbak 21032018
Ambari  use by default the same spark principal for history ui and spark smoke test.
as a consequence the principal is set as spark{principal_suffix}@{realm} and the keytab
is a headless type keytab.

      #configuration
      options.conf['spark.history.ui.acls.enable'] ?= 'true'
      options.conf['spark.history.fs.cleaner.enabled'] ?= 'false'
      options.conf['spark.history.retainedApplications'] ?= '50'
      options.conf['spark.yarn.historyServer.address'] ?= "#{service.node.fqdn}:#{options.conf['spark.history.ui.port']}"
      options.conf['spark.history.kerberos.enabled'] ?= if service.deps.hadoop_core.options.core_site['hadoop.http.authentication.type'] is 'kerberos' then 'true' else 'false'
      
## Wait

      options.wait ?= {}
      options.wait.http =
        host: options.fqdn
        port: options.conf['spark.history.ui.port']
        
## Dependencies

    {merge} = require 'nikita/lib/misc'
