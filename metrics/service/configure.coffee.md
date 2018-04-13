
# Ambari Metrics Configuration

    module.exports = (service) ->
      options = service.options

## Identities

      # Group
      options.group = name: options.group if typeof options.group is 'string'
      options.group ?= {}
      options.group.name ?= 'ams'
      options.group.system ?= true
      # Hadoop Group is also defined in ryba/hadoop/core
      options.hadoop_group = name: options.hadoop_group if typeof options.hadoop_group is 'string'
      options.hadoop_group ?= {}
      options.hadoop_group.name ?= 'hadoop'
      options.hadoop_group.system ?= true
      # User
      options.user ?= {}
      options.user = name: options.user if typeof options.user is 'string'
      options.user.name ?= 'ams'
      options.user.system ?= true
      options.user.gid = options.group.name
      options.user.comment ?= 'Ambari Metrics User'
      options.user.home ?= '/var/lib/ams'
      options.user.groups ?= 'hadoop'
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 32000

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
      options.configurations['ams-hbase-security-site'] ?= {}

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

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

## Ambari Metrics Collector Configuration

      options.collector_hosts ?= []

## Ambari Metrics Monitor Configuration

      options.monitor_hosts ?= []

## Ambari Metrics Grafana Configuration

      options.grafana_hosts ?= []

## Ambari Agent
Register users to ambari agent's user list.

      for srv in service.deps.ambari_agent
        srv.options.users ?= {}
        srv.options.users['ams'] ?= options.user
        srv.options.groups ?= {}
        srv.options.groups['ams'] ?= options.group
  
## Wait

      options.wait = {}
      options.wait_ambari_rest = service.deps.ambari_server.options.wait.rest

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
