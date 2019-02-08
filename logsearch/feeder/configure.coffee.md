
# Ambari Logsearch Feeder Configuration

    module.exports = (service) ->
      options = service.options

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.configurations ?= {}
      options.configurations['logfeeder-env'] ?= {}
      options.configurations['logsearch-common-env'] ?= {}

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      if service.deps.logsearch_service[0].options.solr_external
        if service.deps.logsearch_service[0].options.configurations['logsearch-common-env']['logsearch_external_solr_kerberos_enabled'] is 'true'
          options.configurations['logfeeder-env']['logfeeder_external_solr_kerberos_keytab'] ?= '/etc/security/keytabs/logfeeder.service.keytab'
          options.configurations['logfeeder-env']['logfeeder_external_solr_kerberos_principal'] ?= "logfeeder/_HOST@#{options.krb5.realm}"

## SSL
  
      options.ssl = merge {}, service.deps.ssl.options, options.ssl 
      options.ssl.enabled ?= !!service.deps.ssl
      # options.truststore ?= {}
      if options.ssl.enabled and service.deps.logsearch_service[0].options.configurations['logsearch-common-env']['logsearch_external_solr_ssl_enabled']
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
        throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
        options.configurations['logfeeder-env'] ?= {}
        # options.configurations['logsearch-env']['logsearch_ui_protocol'] ?= 'https'
        # options.configurations['logsearch-env']['logsearch_ui_port'] ?= '61889'
        options.configurations['logfeeder-env']['logfeeder_truststore_location'] ?= '/etc/security/serverKeys/logsearch-feeder-truststore'
        options.configurations['logfeeder-env']['logfeeder_truststore_password'] ?= options.ssl.truststore.password
        options.configurations['logfeeder-env']['logfeeder_keystore_location'] ?= '/etc/security/serverKeys/logsearch-feeder-keystore'
        options.configurations['logfeeder-env']['logfeeder_keystore_password'] ?= options.ssl.keystore.password

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
