
# Ambari Metrics Monitor Configuration

    module.exports = (service) ->
      options = service.options

      options.group = merge service.deps.ambari_server.options.group, options.group
      options.user = merge service.deps.ambari_server.options.user, options.user
      options.test_user = merge service.deps.ambari_server.options.test_user, options.test_user
      options.test_group = merge service.deps.ambari_server.options.test_group, options.test_group

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.conf_dir ?= '/etc/ambari-server/conf'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.master_key ?= null
      options.admin ?= {}
      options.krb5 ?= merge {}, service.deps.ambari_server.options.krb5, options.krb5

## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name

## Ambari Metrics Service Configuration

      for srv in service.deps.metrics_service

        #register host
        srv.options.monitor_hosts ?= []
        srv.options.monitor_hosts.push service.node.fqdn if srv.options.monitor_hosts.indexOf(service.node.fqdn) is -1

## Wait

      options.wait = {}
      options.wait_ambari_rest = service.deps.ambari_server.options.wait.rest

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
