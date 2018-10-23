
# Ambari Logsearch Server Configuration

    module.exports = (service) ->
      options = service.options

      options.group = merge service.deps.smartsense_service[0].options.group, options.group
      options.user = merge service.deps.smartsense_service[0].options.user, options.user
      options.hadoop_group = merge service.deps.smartsense_service[0].options.hadoop_group, options.hadoop_group
      options.test_user = merge service.deps.ambari_server.options.test_user, options.test_user
      options.test_group = merge service.deps.ambari_server.options.test_group, options.test_group

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.conf_dir ?= '/etc/smartsense-activity/conf'
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.krb5 ?= merge {}, service.deps.ambari_server.options.krb5, options.krb5
      options.configurations ?= {}
      options.configurations['activity-zeppelin-site'] ?= {}
      options.download = service.deps.smartsense_service[0].options.download

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## SSL
  
      options.ssl = merge {}, service.deps.ssl.options, options.ssl 
      options.ssl.enabled ?= !!service.deps.ssl
      # options.truststore ?= {}
      throw Error "Required Option: ssl.cert" if  not options.ssl.cert
      throw Error "Required Option: ssl.key" if not options.ssl.key
      throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
      throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
      throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
      options.configurations['activity-zeppelin-site'] ?= {}
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.key.manager.password'] ?= options.ssl.keystore.password
      options.configurations['activity-zeppelin-site']['zeppelin.ssl'] ?= 'true'
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.path'] ?= '/etc/security/serverKeys/activity-explorer-keystore'
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.keystore.password'] ?= options.ssl.keystore.password
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.truststore.path'] ?= '/etc/security/serverKeys/activity-explorer-truststore'
      options.configurations['activity-zeppelin-site']['zeppelin.ssl.truststore.password'] ?= options.ssl.truststore.password

## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover
      options.baremetal = service.deps.ambari_server.options.baremetal

## Ambari Metrics Service Configuration
  
      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v

      for srv in service.deps.smartsense_service

        srv.options.configurations['activity-zeppelin-site'] ?= {}
        
        #register host
        srv.options.server_hosts ?= []
        srv.options.server_hosts.push service.node.fqdn if srv.options.server_hosts.indexOf(service.node.fqdn) is -1

## Wait

      options.wait = {}
      # options.wait.tcp = for srv in service.deps.logsearch_server
      #   host: srv.node.fqdn
      #   port: srv.options?.configurations?['logsearch-env']?['logsearch_ui_port'] or options.configurations['logsearch-env']['logsearch_ui_port']
      options.wait_ambari_rest = service.deps.ambari_server.options.wait.rest

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
