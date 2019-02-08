
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

## Environment

      # Layout
      options.conf_dir ?= '/etc/kafka/conf'
      options.log_dir ?= '/var/log/kafka'
      options.pid_dir ?= '/var/run/kafka'
      options.fqdn ?= service.node.fqdn
      # Opts and Java
      options.java_home ?= service.deps.java.options.java_home
      options.opts ?= ''
      options.heapsize ?= '1024'
      # Misc
      options.fqdn = service.node.fqdn
      options.hostname = service.node.hostname
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.clean_logs ?= false
      options.ranger_admin ?= !!service.deps.ranger_admin

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user

## Identities

      # Group
      options.group = name: options.group if typeof options.group is 'string'
      options.group ?= {}
      options.group.name ?= 'kafka'
      options.group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= options.group.name
      options.user.gid = options.group.name
      options.user.system ?= true
      options.user.comment ?= 'Kafka User'
      options.user.home = "/var/lib/#{options.user.name}"
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 32000
      # Admin
      options.admin ?= {}
      options.admin.principal ?= "#{options.user.name}@#{options.krb5.realm}"
      throw Error "Required Option: admin.password" unless options.admin.password
      #list of kafka superusers
      # match = /^(.+?)[@\/]/.exec options.admin.principal
      # throw Error 'Invalid kafka.broker.admin.principal' unless match
      # options.superusers ?= [match[0]]
      # throw Error 'Kafka admin_principal must be in kafka superusers' unless match[0] in options.superusers
      options.superusers ?= [options.admin.principal.split('@')[0].split('/')[0]]

## Ambari Configurations

      options.configurations ?= {}
      options.configurations['kafka-env'] ?= {}
      # Disbale Atlas Hook by default
      options.configurations['kafka-env']['kafka_log_dir'] ?= options.log_dir #SPEED or COMPRESSION
      options.configurations['kafka-env']['kafka_pid_dir'] ?= options.pid_dir #SPEED or COMPRESSION
      options.configurations['kafka-env']['kafka_user'] ?= options.user.name #on, off
      options.configurations['kafka-env']['kafka_user_nofile_limit'] ?= options.user.limits.nofile #on, off
      options.configurations['kafka-env']['kafka_user_nproc_limit'] ?= options.user.limits.nproc #on, off
      options.configurations['kafka-broker'] ?= {}
      

## Dependencies

    {merge} = require 'nikita/lib/misc'
