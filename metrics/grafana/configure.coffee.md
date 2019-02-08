
# Ambari Metrics Collector Configuration

    module.exports = (service) ->
      options = service.options

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.conf_dir ?= '/etc/ambari-server/conf'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.master_key ?= null
      options.admin ?= {}
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

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
