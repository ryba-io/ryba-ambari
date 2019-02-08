
# HiveServer2 Configuration

The following properties are required by knox in secured mode:

*   hive.server2.enable.doAs
*   hive.server2.allow.user.substitution
*   hive.server2.transport.mode
*   hive.server2.thrift.http.port
*   hive.server2.thrift.http.path

Example:

```json
{ "ryba": {
    "hive": {
      "server2": {
        "heapsize": "4096",
        "opts": "-Dcom.sun.management.jmxremote -Djava.rmi.server.hostname=130.98.196.54 -Dcom.sun.management.jmxremote.rmi.port=9526 -Dcom.sun.management.jmxremote.port=9526 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
      },
      "site": {
        "hive.server2.thrift.port": "10001"
      }
    }
} }
```

    module.exports = (service) ->
      options = service.options

## Identities

      # Hadoop Group
      options.hadoop_group = service.deps.hadoop_core.options.hadoop_group
      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'oozie'
      options.group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'oozie'
      options.user.system ?= true
      options.user.gid ?= 'oozie'
      options.user.comment ?= 'Oozie User'
      options.user.home ?= '/var/lib/oozie'
      options.user.groups ?= 'hadoop'
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 32000

## Environment

      # Layout
      options.conf_dir ?= '/etc/oozie/conf'
      options.data_dir ?= '/var/db/oozie'
      options.log_dir ?= '/var/log/oozie'
      options.pid_dir ?= '/var/run/oozie'
      options.tmp_dir ?= '/var/tmp/oozie'
      # Opts and Java
      options.java_home ?= service.deps.java.options.java_home
      options.heapsize ?= '1024'
      # Misc
      options.fqdn = service.node.fqdn
      options.hostname = service.node.hostname
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.clean_logs ?= false

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

      options.hdfs_krb5_user = service.deps.hdfs[0].options.hdfs.krb5_user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user

## SSL

      options.ssl = merge {}, service.deps.ssl?.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      options.ssl.truststore ?= {}
      if options.ssl.enabled
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        options.ssl.keystore.target = "/etc/security/serverKeys/oozie-keystore"
        throw Error "Required Property: ssl.keystore.password" if not options.ssl.keystore.password
        options.ssl.truststore.target = "/etc/security/serverKeys/oozie-truststore"
        throw Error "Required Property: ssl.truststore.password" if not options.ssl.truststore.password


## Ambari Configurations

      options.configurations ?= {}
      options.configurations['oozie-site'] ?= {}
      options.configurations['oozie-env'] ?= {}
      options.configurations['oozie-env']['oozie_java_home'] ?= options.java_home
      options.configurations['oozie-env']['ssl_keystore_path'] ?= options.ssl.keystore.target
      options.configurations['oozie-env']['ssl_keystore_password'] ?= options.ssl.keystore.password

      
## Dependencies

    {merge} = require 'nikita/lib/misc'
