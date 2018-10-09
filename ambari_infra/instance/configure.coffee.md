
# Ambari Infra Instance Configuration

    module.exports = (service) ->
      options = service.options

      options.group = merge service.deps.ambari_infra_service[0].options.group, options.group
      options.user = merge service.deps.ambari_infra_service[0].options.user, options.user
      options.hadoop_group = merge service.deps.ambari_infra_service[0].options.hadoop_group, options.hadoop_group
      options.test_user = merge service.deps.ambari_server.options.test_user, options.test_user
      options.test_group = merge service.deps.ambari_server.options.test_group, options.test_group

## Environment

      options.fqdn = service.node.fqdn
      # options.http ?= '/var/www/html'
      options.sudo ?= false
      options.iptables ?= service.deps.iptables and service.deps.iptables.options.action is 'start'
      options.krb5 ?= merge {}, service.deps.ambari_server.options.krb5, options.krb5
      options.configurations ?= {}
      options.configurations['infra-solr-env'] ?= {}
      options.configurations['infra-solr-env']['infra_solr_port'] ?= '8886'

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      options.configurations['infra-solr-env']['infra_solr_kerberos_keytab'] ?= "/etc/security/keytabs/ambari-infra-solr.service.keytab"
      options.configurations['infra-solr-env']['infra_solr_kerberos_principal'] ?= "#{options.user.name}/_HOST@#{options.krb5.realm}"
      options.configurations['infra-solr-env']['infra_solr_web_kerberos_keytab'] ?= "/etc/security/keytabs/spnego.service.keytab"
      options.configurations['infra-solr-env']['infra_solr_web_kerberos_principal'] ?= "HTTP/_HOST@#{options.krb5.realm}"

## SSL
  
      options.ssl = merge {}, service.deps.ssl.options, options.ssl 
      options.ssl.enabled ?= !!service.deps.ssl
      # options.truststore ?= {}
      if options.ssl.enabled
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Property: keystore.password" if not options.ssl.keystore.password
        throw Error "Required Property: truststore.password" if not options.ssl.truststore.password
        options.configurations['infra-solr-env']['infra_solr_ssl_enabled'] ?= 'Yes'
        options.configurations['infra-solr-env']['infra_solr_truststore_location'] ?= '/etc/security/serverKeys/infra.solr.truststore.jks'
        options.configurations['infra-solr-env']['infra_solr_truststore_password'] ?= options.ssl.truststore.password
        options.configurations['infra-solr-env']['infra_solr_keystore_location'] ?= '/etc/security/serverKeys/infra.solr.keystore.jks'
        options.configurations['infra-solr-env']['infra_solr_keystore_password'] ?= options.ssl.keystore.password


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

      for srv in service.deps.ambari_infra_service

        srv.options.configurations['infra-solr-env'] ?= {}
        enrich_config options.configurations['infra-solr-env'], srv.options.configurations['infra-solr-env']
        #register host
        srv.options.instance_hosts ?= []
        srv.options.instance_hosts.push service.node.fqdn if srv.options.instance_hosts.indexOf(service.node.fqdn) is -1

## Wait

      options.wait = {}
      options.wait_ambari_rest = service.deps.ambari_server.options.wait.rest

## Dependencies

    url = require 'url'
    {merge} = require 'nikita/lib/misc'
