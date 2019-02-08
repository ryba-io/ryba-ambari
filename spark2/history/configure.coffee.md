
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
      options.iptables ?= !!service.deps.iptables and service.deps.iptables?.action is 'start'
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

## HDFS Log Dir
HDFS Directory from which the spark history server will load jobs information once in state finished.

      options.conf['spark.eventLog.dir'] ?= "#{service.deps.hdfs_nn[0].options.core_site['fs.defaultFS']}/user/#{options.user.name}2/applicationHistory"
      options.conf['spark.history.fs.logDirectory'] ?= options.conf['spark.eventLog.dir']
## SSL

      options.ssl = merge {}, service.deps.ssl.options, service.deps.spark_local.options.ssl
      options.conf[prop] ?= service.deps.spark_local.options.conf[prop] for prop in [
        'spark.ssl.enabled'
        'spark.ssl.enabledAlgorithms'
        'spark.ssl.keyPassword'
        'spark.ssl.keyStore'
        'spark.ssl.keyStorePassword'
        'spark.ssl.protocol'
        'spark.ssl.trustStore'
        'spark.ssl.trustStorePassword'
      ]
      if options.conf['spark.ssl.enabled'] is 'true'
        options.conf['spark.ui.https.enabled'] ?= true
        options.conf['spark.ssl.ui.port'] ?= '18082'

## Port
Quoting  [spark documentation](https://spark.apache.org/docs/2.2/configuration.html):
When not set, the SSL port will be derived from the non-SSL port for the same service.
 A value of "0" will make the service bind to an ephemeral port. 

      options.conf['spark.history.ui.port'] ?= '18081'

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
      options.conf['spark.history.kerberos.principal'] ?= service.deps.spark[0].options.krb5.principal
      options.conf['spark.history.kerberos.keytab'] ?= service.deps.spark[0].options.krb5.keytab
      
## Wait

      options.wait ?= {}
      options.wait.http =
        host: options.fqdn
        port: options.conf['spark.history.ui.port']
        
## Dependencies

    {merge} = require 'nikita/lib/misc'
