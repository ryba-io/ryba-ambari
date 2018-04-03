
# Ambari Agent Configuration

    module.exports = (service) ->
      options = service.options

## Environment

      options.group = merge service.deps.ambari_server_takeover[0].options.group, options.group
      options.user = merge service.deps.ambari_server_takeover[0].options.user, options.user
      options.test_user = merge service.deps.ambari_server_takeover[0].options.test_user, options.test_user
      options.test_group = merge service.deps.ambari_server_takeover[0].options.test_group, options.test_group
      options.fqdn = service.node.fqdn

## Ambari Rest Api URL

      options.ambari_url ?= service.deps.ambari_server[0].options.ambari_url
      options.ambari_admin_password ?= service.deps.ambari_server[0].options.admin_password

## Kerberos

      options.krb5_enabled ?= service.deps.ambari_server[0].options.krb5_enabled
      if options.krb5_enabled
        options.krb5 ?= {}
        options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
        throw Error 'Required Options: "realm"' unless options.krb5.realm
        options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
        # Krb5 Validation
        throw Error "Require Property: krb5.admin.kadmin_principal" unless options.krb5.admin.kadmin_principal
        throw Error "Require Property: krb5.admin.kadmin_password" unless options.krb5.admin.kadmin_password
        throw Error "Require Property: krb5.admin.admin_server" unless options.krb5.admin.admin_server

## Ambari TakeOver Configuration

      options.cluster_name ?= service.deps.ambari_server_takeover[0].options.cluster_name

### User Provisionning
Contains object of user that ambari-agent should create on all hosts. By default
Ambari needs to all user on all node even if the service is not installed on a host.

The components should register their user to ambari agents

      options.users ?= {}
      options.groups ?= {}

## Config Groups
      
      options.config_groups ?= []
      for srv in service.deps.ambari_server_takeover
        for name in options.config_groups
          srv.options.config_groups ?= {}
          srv.options.config_groups[name] ?= {}
          srv.options.config_groups[name]['hosts'] ?= []
          srv.options.config_groups[name]['hosts'].push service.node.fqdn unless srv.options.config_groups[name]['hosts'].indexOf(service.node.fqdn) > -1
          
## Wait Ambari

      options.wait_ambari_rest = service.deps.ambari_server[0].options.wait.rest


## Dependencies

    {merge} = require 'nikita/lib/misc'
