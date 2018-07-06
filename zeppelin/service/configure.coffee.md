
# Ambari Logsearch Configuration

    module.exports = (service) ->
      options = service.options

## Identities

      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'zeppelin'
      options.group.system ?= true

      # Hadoop Group is also defined in ryba/hadoop/core
      options.hadoop_group = name: options.hadoop_group if typeof options.hadoop_group is 'string'
      options.hadoop_group ?= {}
      options.hadoop_group.name ?= 'hadoop'
      options.hadoop_group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'zeppelin'
      options.user.system ?= true
      options.user.gid = options.group.name
      options.user.comment ?= 'Ambari Zeppelin User'
      options.user.home ?= '/var/run/zeppelin'
      options.user.groups ?= 'hadoop'
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 32000
      
      options.livy_ssl_enabled ?= (service.deps.spark_livy_server.length > 0) and service.deps.spark_livy_server[0].options.ssl.enabled
      # options.group = merge service.deps.ambari_server.options.group, options.group
      # options.user = merge service.deps.ambari_server.options.user, options.user
      # options.test_user = merge service.deps.ambari_server.options.test_user, options.test_user
      # options.test_group = merge service.deps.ambari_server.options.test_group, options.test_group

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      # options.conf_dir ?= '/etc/ambari-server/conf'
      options.sudo ?= false
      options.admin ?= {}
      options.krb5 ?= merge {}, service.deps.ambari_server.options.krb5, options.krb5
      options.configurations ?= {}

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      options.krb5.principal ?= "zeppelin@#{options.krb5.realm}"
      options.krb5.keytab ?= '/etc/security/keytabs/zeppelin.server.kerberos.principal'
      # options.krb5.principal ?= "spark/#{service.node.fqdn}@#{options.krb5.realm}"
      # options.krb5.keytab ?= '/etc/security/keytabs/spark.service.keytab'
      throw Error 'Required Options: "realm"' unless options.krb5.realm
      throw Error 'Required Options: "password"' unless options.krb5.password
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      
      options.identities ?= {}
      options.identities['zeppelin_master'] ?= {}
      options.identities['zeppelin_master']['principal'] ?= {}
      options.identities['zeppelin_master']['principal']['configuration'] ?= 'zeppelin-env/zeppelin.server.kerberos.principal'
      options.identities['zeppelin_master']['principal']['type'] ?= 'user'
      options.identities['zeppelin_master']['principal']['local_username'] ?= options.user.name
      options.identities['zeppelin_master']['principal']['value'] ?= options.krb5.principal #options.spark.krb5_user.principal
      options.identities['zeppelin_master']['name'] ?= 'zeppelin_user'
      options.identities['zeppelin_master']['keytab'] ?= {}
      options.identities['zeppelin_master']['keytab']['owner'] ?= {}
      options.identities['zeppelin_master']['keytab']['owner']['access'] ?= 'r' 
      options.identities['zeppelin_master']['keytab']['owner']['name'] ?= options.user.name 
      options.identities['zeppelin_master']['keytab']['group'] ?= {}
      options.identities['zeppelin_master']['keytab']['group']['access'] ?= 'r'
      options.identities['zeppelin_master']['keytab']['group']['name'] ?= options.hadoop_group.name
      options.identities['zeppelin_master']['keytab']['file'] ?= options.krb5.keytab
      options.identities['zeppelin_master']['keytab']['configuration'] ?= 'zeppelin-env/zeppelin.server.kerberos.keytab'

## Configurations

      options.configurations ?= {}
      options.configurations['zeppelin-config'] ?= {}

## Ambari REST API

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

## Ambari Zeppelin Configuration

      options.configurations['zeppelin-env'] ?= {}
      
## Ambari Zeppelin Master Configuration

      options.master_hosts ?= []

## Ambari Agent
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['zeppelin'] ?= options.user
        srv.options.groups ?= {}
        srv.options.groups['zeppelin'] ?= options.group
      
## Wait

      options.wait = {}
      options.wait_ambari_rest = service.deps.ambari_server.options.wait.rest

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
