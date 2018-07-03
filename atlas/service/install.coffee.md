
# Ambari Takeover

    module.exports = header: 'HBase Ambari Install', handler: (options) ->
      
## Register

      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_update'], "ryba-ambari-actions/lib/hosts/component_update"
      @registry.register ['ambari','configs','groups_add'], 'ryba-ambari-actions/lib/configs/groups/add'
      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'

## ATLAS Configuration


      @call header: 'KafKa Topic ACL (Ranger)', if: options.ranger_kafka_install?, ->
        @ranger_service_wait
          header: "Kafka Plugin Wait"
          username: options.ranger_admin.options.admin.username
          password: options.ranger_admin.options.admin.password
          url: options.ranger_admin.options.install['policymgr_external_url']
          service: options.kafka_policy.service
        @ranger_user
          header: "Ranger admin"
          username: options.ranger_admin.options.admin.username
          password: options.ranger_admin.options.admin.password
          url: options.ranger_admin.options.install['policymgr_external_url']
          user: options.ranger_user
        @ranger_policy
          header: 'Kafka Plugin ACL'
          username: options.ranger_admin.options.admin.username
          password: options.ranger_admin.options.admin.password
          url: options.ranger_admin.options.install['policymgr_external_url']
          policy: options.kafka_policy
        @wait
          time: 10000
          if: -> @status -1

      # @ambari.configs.default
      #   header: 'ATLAS Configuration'
      #   url: options.ambari_url
      #   if: options.post_component and options.takeover
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   cluster_name: options.cluster_name
      #   stack_name: options.stack_name
      #   stack_version: options.stack_version
      #   discover: true
      #   configurations: options.configurations
      #   target_services: 'ATLAS'
      # @ambari.configs.update
      #   header: 'Update application-properties'
      #   if: options.takeover or options.baremetal
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   config_type: 'application-properties'
      #   cluster_name: options.cluster_name
      #   properties: options.configurations['application-properties']

      # @ambari.configs.update
      #   header: 'Upload ranger-atlas-plugin-properties'
      #   if : options.post_component and (options.takeover or options.baremetal)
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   config_type: 'ranger-atlas-plugin-properties'
      #   cluster_name: options.cluster_name
      #   properties: options.configurations['ranger-atlas-plugin-properties']
      
      # 
      # @ambari.configs.update
      #   header: 'Upload ranger-atlas-security'
      #   if : options.post_component and (options.takeover or options.baremetal)
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   config_type: 'ranger-atlas-security'
      #   cluster_name: options.cluster_name
      #   properties: options.configurations['ranger-atlas-security']
      # 
      # @ambari.configs.update
      #   header: 'Upload ranger-atlas-policymgr-ssl'
      #   if : options.post_component and (options.takeover or options.baremetal)
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   config_type: 'ranger-atlas-policymgr-ssl'
      #   cluster_name: options.cluster_name
      #   properties: options.configurations['ranger-atlas-policymgr-ssl']
      # 
      # @ambari.configs.update
      #   header: 'Upload ranger-atlas-audit'
      #   if : options.post_component and (options.takeover or options.baremetal)
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   config_type: 'ranger-atlas-audit'
      #   cluster_name: options.cluster_name
      #   properties: options.configurations['ranger-atlas-audit']
      @call -> process.exit 1

      @ambari.services.add
        header: 'ATLAS Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ATLAS'

      @ambari.services.wait
        header: 'ATLAS Service WAITED'
        if: options.post_component and options.takeover
        username: 'admin'
        url: options.ambari_url
        cluster_name: options.cluster_name
        password: options.ambari_admin_password
        name: 'ATLAS'

      @ambari.services.component_add
        header: 'ATLAS Add'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ATLAS_METADATA_SERVER'
        service_name: 'ATLAS'


## Dependencies

    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
