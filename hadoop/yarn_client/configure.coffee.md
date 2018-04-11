

# YARN Client Configure

    module.exports = (service) ->
      options = service.options

## Environment

      options.log_dir ?= '/var/log/hadoop-yarn'
      # options.pid_dir ?= '/var/run/hadoop-yarn'
      options.conf_dir ?= service.deps.hadoop_core.options.conf_dir
      options.opts ?= ''
      options.heapsize ?= '1024m'
      options.home ?= '/usr/hdp/current/hadoop-yarn-client'
      # Misc
      options.java_home ?= service.deps.java.options.java_home

## Identities

      options.group = merge {}, service.deps.hadoop_core.options.yarn.group, options.group
      options.user = merge {}, service.deps.hadoop_core.options.yarn.user, options.user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user
      options.fqdn ?= service.node.fqdn

## Configuration

      options.yarn_site ?= {}

## Wait

      options.wait_yarn_ts = service.deps.yarn_ts[0].options.wait
      options.wait_yarn_rm = service.deps.yarn_rm[0].options.wait

      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v
          
      for srv in service.deps.yarn
        #add hosts
        srv.options.client_hosts ?= []
        srv.options.client_hosts.push options.fqdn if srv.options.client_hosts.indexOf(options.fqdn) is -1

## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover

## Dependencies

    {merge} = require 'nikita/lib/misc'
