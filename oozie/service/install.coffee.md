
# Oozie Service Install

    module.exports =  header: 'Ambari Oozie Service Install', handler: (options) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'
      @registry.register ['ambari','configs','groups_add'], 'ryba-ambari-actions/lib/configs/groups/add'

## Kerberos Descriptor

      @ambari.kerberos.descriptor.update
        header: 'Kerberos Artifact'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        stack_name: options.stack_name
        stack_version: options.stack_version
        cluster_name: options.cluster_name
        identities: options.identities['oozie']
        service: 'OOZIE'
        source: 'COMPOSITE'

## Render Oozie-env file

      writes = [
          match: /^export OOZIE_HTTPS_KEYSTORE_FILE=.*$/mg
          replace: "export OOZIE_HTTPS_KEYSTORE_FILE=#{options.ssl.keystore.target}"
          append: true
        ,
          match: /^export OOZIE_HTTPS_KEYSTORE_PASS=.*$/mg
          replace: "export OOZIE_HTTPS_KEYSTORE_PASS=#{options.ssl.keystore.password}"
          append: true
        ,
          match: /^export CATALINA_OPTS="${CATALINA_OPTS} -Djavax.net.ssl.trustStore=(.*)/m
          replace: """
          export CATALINA_OPTS="${CATALINA_OPTS} -Djavax.net.ssl.trustStore=#{options.ssl.truststore.target}"
          """
          append: true
        ,
          match: /^export CATALINA_OPTS="${CATALINA_OPTS} -Djavax.net.ssl.trustStorePassword=(.*)/m
          replace: """
          export CATALINA_OPTS="${CATALINA_OPTS} -Djavax.net.ssl.trustStorePassword=#{options.ssl.truststore.password}"
          """
          append: true
        ,
          match: /^export OOZIE_CLIENT_OPTS="${OOZIE_CLIENT_OPTS} -Doozie.connection.retry.count=5 -Djavax.net.ssl.trustStore=(.*)/m
          replace: """
          export OOZIE_CLIENT_OPTS="${OOZIE_CLIENT_OPTS} -Doozie.connection.retry.count=5 -Djavax.net.ssl.trustStore=#{options.ssl.truststore.target}"
          """
          append: true
        ,
          match: /^export OOZIE_CLIENT_OPTS="${OOZIE_CLIENT_OPTS} -Doozie.connection.retry.count=5 -Djavax.net.ssl.trustStorePassword=(.*)/m
          replace: """
          export OOZIE_CLIENT_OPTS="${OOZIE_CLIENT_OPTS} -Doozie.connection.retry.count=5 -Djavax.net.ssl.trustStorePassword=#{options.ssl.truststore.password}"
          """
          append: true
        
        ]
      @file.assert
        target: options.hadoop_lib_home
        filetype: 'directory'
      @file.render
        if: options.post_component
        header: 'Render oozie-env'
        target: "#{options.cache_dir}/oozie-env.sh"
        source: "#{__dirname}/../resources/oozie-env.sh.j2"
        local: true
        context: options.configurations['oozie-env']
        write: writes
        ssh: false
        backup: true
      @call
        if: options.post_component
      , (_, cb) ->
        ssh2fs.readFile null, "#{options.cache_dir}/oozie-env.sh", (err, content) =>
          try
            throw err if err
            content = content.toString()
            options.configurations['oozie-env']['content'] ?= content
            cb()
          catch err
            cb err


## Upload Default Configuration

      @ambari.configs.default
        header: 'OOZIE Configuration'
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        discover: true
        configurations: options.configurations
        target_services: 'OOZIE'

### OOZIE Service
      
      @ambari.services.add
        header: 'OOZIE Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'OOZIE'


## Add and enable OOZIE component
add `OOZIE_SERVER`, `OOZIE_CLIENT`.

      @ambari.services.wait
        header: 'OOZIE Service WAITED'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'OOZIE'

      @ambari.services.component_add
        if: options.post_component and options.takeover
        header: 'OOZIE_SERVER'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'OOZIE_SERVER'
        service_name: 'OOZIE'

      @ambari.services.component_add
        if: options.post_component and options.takeover
        header: 'OOZIE_CLIENT'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'OOZIE_CLIENT'
        service_name: 'OOZIE'

    
      for host in options.server_hosts
        @ambari.hosts.component_add
          header: 'OOZIE_SERVER'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'OOZIE_SERVER'
          hostname: host
      
      for host in options.client_hosts
        @ambari.hosts.component_add
          header: 'OOZIE_CLIENT'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'OOZIE_CLIENT'
          hostname: host

## Ambari Config Groups
          
      # @call
      #   if: options.post_component and options.takeover
      # , ->
      #   if (options.server_hosts.length > 1) and options.ssl.enabled
      #     @each options.server_hosts, (opts, next) ->
      #       host = opts.key
      #       protocol = if options.ssl.enabled then 'https://' else 'http://'
      #       group_name = "oozie_ha_env_ssl_#{host.split(".")[0]}"
      #       writes.push
      #         match: /^export OOZIE_BASE_URL=.*$/mg
      #         replace: "export OOZIE_BASE_URL=\"#{protocol}#{host}:#{options.http_port}/oozie\""
      #       @file.render
      #         debug: true
      #         if: options.post_component
      #         header: "Render oozie-env + #{host}"
      #         target: "#{options.cache_dir}/oozie-env-#{host}.sh"
      #         source: "#{__dirname}/../resources/oozie-env.sh.j2"
      #         local: true
      #         context: options.configurations['oozie-env']
      #         write: writes
      #         ssh: false
      #         backup: true
      #       @call
      #         if: options.post_component
      #       , (_, cb) ->
      #         ssh2fs.readFile null, "#{options.cache_dir}/oozie-env-#{host}.sh", (err, content) =>
      #           try
      #             throw err if err
      #             content = content.toString()
      #             @ambari.configs.groups_add
      #               header: "#{group_name}"
      #               url: options.ambari_url
      #               username: 'admin'
      #               password: options.ambari_admin_password
      #               cluster_name: options.cluster_name
      #               group_name: group_name
      #               tag: group_name
      #               description: "#{group_name} config groups"
      #               hosts: host
      #               desired_configs: 
      #                 type: 'oozie-env'
      #                 tag: group_name
      #                 properties: content: content
      #             @next cb
      #           catch err
      #             console.log err
      #             cb
      #       @next next
                # catch err
                #   cb err
        # @each options.config_groups, (opts, cb) ->
        #   {key, value} = opts
        #   @ambari.configs.groups_add
        #     header: "#{key}"
        #     url: options.ambari_url
        #     username: 'admin'
        #     password: options.ambari_admin_password
        #     cluster_name: options.cluster_name
        #     group_name: key
        #     tag: key
        #     description: "#{key} config groups"
        #     hosts: value.hosts
        #     desired_configs: 
        #       type: value.type
        #       tag: value.tag
        #       properties: value.properties
        #   @next cb

## Dependencies

    path = require 'path'
    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
