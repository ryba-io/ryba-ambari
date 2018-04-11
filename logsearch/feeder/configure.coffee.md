
# Ambari Logsearch Feeder Configuration

    module.exports = (service) ->
      options = service.options

      options.group = merge service.deps.logsearch_service[0].options.group, options.group
      options.user = merge service.deps.logsearch_service[0].options.user, options.user
      options.hadoop_group = merge service.deps.logsearch_service[0].options.hadoop_group, options.hadoop_group
      options.test_user = merge service.deps.ambari_server.options.test_user, options.test_user
      options.test_group = merge service.deps.ambari_server.options.test_group, options.test_group

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.krb5 ?= merge {}, service.deps.ambari_server.options.krb5, options.krb5
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


## Ambari

      #ambari server configuration
      options.post_component = service.instances[0].node.fqdn is service.node.fqdn
      options.ambari_host = service.node.fqdn is service.deps.ambari_server.node.fqdn
      options.ambari_url ?= service.deps.ambari_server.options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server.options.ambari_admin_password
      options.cluster_name ?= service.deps.ambari_server.options.cluster_name
      options.takeover = service.deps.ambari_server.options.takeover

## Ambari Metrics Service Configuration

      enrich_config = (source, target) ->
        for k, v of source
          target[k] ?= v

      for srv in service.deps.logsearch_service

        srv.options.configurations['logsearch-env'] ?= {}
        srv.options.configurations['logfeeder-env'] ?= {}
        enrich_config options.configurations['logfeeder-env'], srv.options.configurations['logfeeder-env']
        enrich_config options.configurations['logsearch-common-env'], srv.options.configurations['logsearch-common-env']
        #register host
        srv.options.feeder_hosts ?= []
        srv.options.feeder_hosts.push service.node.fqdn if srv.options.feeder_hosts.indexOf(service.node.fqdn) is -1

## Wait

      options.wait = {}
      options.wait_ambari_rest = service.deps.ambari_server.options.wait.rest
      options.wait_logsearch_server = service.deps.logsearch_server[0].options.wait

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
