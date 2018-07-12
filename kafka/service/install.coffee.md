
# KAFKA Service Install

    module.exports =  header: 'Ambari KAFKA Service Install', handler: (options) ->
      
## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_update'], "ryba-ambari-actions/lib/hosts/component_update"
      @registry.register ['ambari','configs','groups_add'], 'ryba-ambari-actions/lib/configs/groups/add'
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

## Upload Default Configuration

      KAFKA_HEAP_OPTS = options.env['KAFKA_HEAP_OPTS']
      @file.render
        header: 'Render spark-env'
        target: "#{options.cache_dir}/kafka-env.sh"
        source: "#{__dirname}/../resources/kafka-env.sh.ambari.j2"
        local: true
        context:
          KAFKA_HEAP_OPTS: KAFKA_HEAP_OPTS
        backup: true
        ssh:false
      @call (opts, cb) ->
        ssh2fs.readFile null, "#{options.cache_dir}/kafka-env.sh", (err, content) =>
          try
            throw err if err
            content = content.toString()
            options.configurations['kafka-env'] ?= {}
            options.configurations['kafka-env']['content'] = content
            cb()
          catch err
            cb err


## Kerberos Descriptor Artifact

      @ambari.kerberos.descriptor.update
        header: 'Kerberos Artifact Update'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        stack_name: options.stack_name
        stack_version: options.stack_version
        cluster_name: options.cluster_name
        source: 'COMPOSITE'
        service: 'KAFKA'
        component: 'KAFKA_BROKER'
        identities: options.identities['kafka']

      @ambari.configs.default
        header: 'KAFKA Configuration'
        url: options.ambari_url
        if: options.post_component
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        discover: true
        configurations: options.configurations
        target_services: 'KAFKA'


## kafka related properties

      @ambari.configs.update
        header: 'Upload kafka-broker'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'kafka-broker'
        cluster_name: options.cluster_name
        properties: options.configurations['kafka-broker']

      @ambari.configs.update
        header: 'Upload kafka-env'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'kafka-env'
        cluster_name: options.cluster_name
        properties: options.configurations['kafka-env']

## kafka Log4j

      @file.properties
        header: 'Broker Log4j'
        if: options.post_component and options.takeover
        target: "#{options.cache_dir}/kafka-log4j.properties"
        content: options.log4j.properties
        ssh: false
        backup: true
      @call
        header: 'Upload kafka-logj4'
        if: options.post_component
      , (_, callback) ->
          console.log 'TODO put baremetal option on this config'
          ssh2fs.readFile null, "#{options.cache_dir}/kafka-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations['kafka-log4j'] = merge {}, options.configurations['kafka-log4j'], content: content
              callback()
            catch err
              callback err


## Upload Default Configuration

      # @call -> console.log options.configurations
      @ambari.configs.default
        header: 'KAFKA Configuration'
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        discover: true
        configurations: options.configurations
        target_services: 'KAFKA'

## RANGER PLUGIN Properties

      @ambari.configs.update
        header: 'Upload ranger-kafka-plugin-properties'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-kafka-plugin-properties'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-kafka-plugin-properties']

      @ambari.configs.update
        header: 'Upload ranger-kafka-security'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-kafka-security'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-kafka-security']

      @ambari.configs.update
        header: 'Upload ranger-kafka-policymgr-ssl'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-kafka-policymgr-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-kafka-policymgr-ssl']

      @ambari.configs.update
        header: 'Upload ranger-kafka-audit'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-kafka-audit'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-kafka-audit']

## Add KAFKA Service

      @ambari.services.add
        header: 'KAFKA Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'KAFKA'

## Add and enable KAFKA component
add `KAFKA_BROKER` components to cluster in `INIT` state.

      @ambari.services.wait
        header: 'KAFKA Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'KAFKA'

      @ambari.services.component_add
        if: options.post_component and options.takeover
        header: 'KAFKA_BROKER ADD'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'KAFKA_BROKER'
        service_name: 'KAFKA'

      for host in options.broker_hosts
        @ambari.hosts.component_add
          header: "KAFKA_BROKER #{host}"
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'KAFKA_BROKER'
          hostname: host

## Ambari Config Groups
          
      @call
        if: options.post_component and options.takeover
      , ->
        @each options.config_groups, (opts, cb) ->
          {key, value} = opts
          @ambari.configs.groups_add
            header: "#{key}"
            url: options.ambari_url
            username: 'admin'
            password: options.ambari_admin_password
            cluster_name: options.cluster_name
            group_name: key
            tag: key
            description: "#{key} config groups"
            hosts: value.hosts
            desired_configs: 
              type: value.type
              tag: value.tag
              properties: value.properties
          @next cb


## Dependencies

    path = require 'path'
    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
