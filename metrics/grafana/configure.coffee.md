
# Ambari Metrics Collector Configuration

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
      options.configurations ?= {}

## Kerberos


      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]

## SSL

      options.ssl = merge {}, service.deps.ssl?.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      if options.ssl.enabled
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        throw Error "Required Option: ssl.key" if not options.ssl.key

## Server Properties

      options.ini ?= {}
      #webui
      options.ini['server'] ?= {}
      options.ini['server']['http_port'] ?= '3000'
      options.ini['server']['domain'] ?= service.node.fqdn

## Grafana Ini

      options.configurations['ams-grafana-ini'] ?= {}
      options.configurations['ams-grafana-ini']['port'] ?= '3000'
      options.configurations['ams-grafana-ini']['protocol'] ?= if options.ssl.enabled then 'https' else 'http'
      options.configurations['ams-grafana-ini']['cert_file'] ?= "/etc/security/certs/grafana_cert.pem"
      options.configurations['ams-grafana-ini']['cert_key'] ?= "/etc/security/certs/grafana_key.pem"

## Grafana Env

      options.configurations['ams-grafana-env'] ?= {}
      throw Error 'Undefined metrics_grafana_password ' unless options.metrics_grafana_password?
      options.configurations['ams-grafana-env']['metrics_grafana_password'] ?= options.metrics_grafana_password


## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover

## Ambari Metrics Service Enrich

      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v


      for srv in service.deps.metrics_service

        srv.options.configurations['ams-grafana-env'] ?= {}
        srv.options.configurations['ams-grafana-ini'] ?= {}
        enrich_config options.configurations['ams-grafana-env'], srv.options.configurations['ams-grafana-env']
        enrich_config options.configurations['ams-grafana-ini'], srv.options.configurations['ams-grafana-ini']
        #register host
        srv.options.grafana_hosts ?= []
        srv.options.grafana_hosts.push service.node.fqdn if srv.options.grafana_hosts.indexOf(service.node.fqdn) is -1

## Wait

      options.wait = {}
      options.wait_ambari_rest = service.deps.ambari_server.options.wait.rest

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
