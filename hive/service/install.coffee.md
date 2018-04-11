
# Hive Service Install

    module.exports =  header: 'Ambari Hive Service Install', handler: (options) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      
## WEBHCAT Environment

      @call header: 'Render wehbcat-env', ->
        webhcat_opts = ''
        webhcat_opts += " -D#{k}=#{v}" for k, v of options.webhcat_opts.java_properties
        webhcat_opts += " #{k}#{v}" for k, v of options.webhcat_opts.jvm
        @file
          source: "#{__dirname}/../resources/webhcat-env.sh.j2"
          local: true
          target: "#{options.cache_dir}/webhcat-env.sh"
          ssh: false
          mode: 0o0755
          write: [
            match: RegExp "export HADOOP_OPTS=.*", 'm'
            replace: "export HADOOP_OPTS=\"${HADOOP_OPTS} #{webhcat_opts}\" # RYBA, DONT OVERWRITE"
            append: true
          ]
      @call
        header: 'Upload webhcat-env'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/webhcat-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations['webhcat-env'] =  merge {},  options.configurations['webhcat-env'], content: content
              callback()
            catch err
              callback err

## WEBHCAT Log4j Configuration

      @file
        header: 'Render webhcat-log4j'
        if: options.post_component and options.webhcat_log4j?
        target: "#{options.cache_dir}/webhcat-log4j.properties"
        source: "#{__dirname}/../resources/webhcat-log4j.properties"
        local: true
        ssh: false
        write: for k, v of options.webhcat_log4j
          match: RegExp "#{k}=.*", 'm'
          replace: "#{k}=#{v}"
          append: true
      @call
        header: 'Upload webhcat-log4j'
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/webhcat-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations['webhcat-env'] ?= content:  content
              callback()
            catch err
              callback err

## HCAT Environment

      @call
        header: 'Upload hcat-env'
        if: options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/hcat-env.sh.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations['hcat-env'] =  merge {},  options.configurations['hcat-env'], content: content
              callback()
            catch err
              callback err

## HIVE Env

      @call
        header: 'Upload hive-env'
        if: options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/hive-env.sh.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations['hive-env'] =  merge {},  options.configurations['hive-env'], content: content
              callback()
            catch err
              callback err

## Upload Default Configuration

      # @call -> console.log options.configurations
      @ambari.configs.default
        header: 'HIVE Configuration'
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        discover: true
        configurations: options.configurations
        target_services: 'HIVE'


# ## HIVE Exec-Log4j
# 
#       @file.render
#         header: 'Render hive-exec-log4j2'
#         if: options.post_component and options.takeover
#         source: "#{__dirname}/../resources/hive-exec-log4j.properties.j2"
#         local: true
#         target: "#{options.cache_dir}/hive-exec-log4j.properties"
#         context: options
#         ssh: false
#       @call
#         header: 'Upload hive-exec-log4j2'
#         if: options.post_component and options.takeover
#       , (_, callback) ->
#           ssh2fs.readFile null, "#{options.cache_dir}/hive-exec-log4j.properties", (err, content) =>
#             try
#               throw err if err
#               content = content.toString()
#               options.configurations['hive-exec-log4j2'] =  merge {},  options.configurations['hive-exec-log4j2'], content: content
#               callback()
#             catch err
#               callback err
# 
# ## HIVE Log4j
# 
#       @file.properties
#         header: 'Render hive-log4j2'
#         if: options.post_component and options.takeover
#         target: "#{options.cache_dir}/hive-log4j.properties"
#         content: options.hive_log4j
#         backup: true
#         ssh: false
# 
#       @call
#         header: 'Upload hive-log4j2'
#         if: options.post_component
#       , (_, callback) ->
#           ssh2fs.readFile null, "#{options.cache_dir}/hive-log4j.properties", (err, content) =>
#             try
#               throw err if err
#               content = content.toString()
#               options.configurations['hive-log4j2'] =  merge {},  options.configurations['hive-log4j2'], content: content
#               callback()
#             catch err
#               callback err

## RANGER PLUGIN Properties

      @ambari.configs.update
        header: 'Upload ranger-hive-plugin-properties'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-plugin-properties'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-plugin-properties']

      @ambari.configs.update
        header: 'Upload ranger-hive-security'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-security'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-security']

      @ambari.configs.update
        header: 'Upload ranger-hive-policymgr-ssl'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-policymgr-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-policymgr-ssl']

      @ambari.configs.update
        header: 'Upload ranger-hive-audit'
        if : options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-audit'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-audit']

## Add HIVE Service

      @ambari.services.add
        header: 'HIVE Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE'

## Add and enable HIVE component
add `HIVE_SERVER`, `HCAT`, `HIVE_CLIENT`, `HIVE_METASTORE` (LLAP)
 `WEBHCAT_SERVER` components to cluster in `INIT` state.

      @ambari.services.wait
        header: 'HIVE Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE'

      @ambari.services.component_add
        if: options.post_component and options.takeover
        header: 'HIVE_SERVER'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_SERVER'
        service_name: 'HIVE'

      @ambari.services.component_add
        if: options.post_component and options.takeover
        header: 'HCAT'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HCAT'
        service_name: 'HIVE'
        
      @ambari.services.component_add
        if: options.post_component and options.takeover
        header: 'HIVE_CLIENT'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_CLIENT'
        service_name: 'HIVE'

      # @ambari.services.component_add
      #   if: options.post_component
      #   header: 'HCAT_CLIENT'
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   cluster_name: options.cluster_name
      #   component_name: 'HCAT_CLIENT'
      #   service_name: 'HIVE'
        
      @ambari.services.component_add
        if: options.post_component and options.takeover
        header: 'HIVE_METASTORE'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_METASTORE'
        service_name: 'HIVE'

      @ambari.services.component_add
        if: options.post_component and options.takeover
        header: 'WEBHCAT_SERVER'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'WEBHCAT_SERVER'
        service_name: 'HIVE'

      for host in options.server2_hosts
        @ambari.hosts.component_add
          header: 'HIVE_SERVER'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_SERVER'
          hostname: host

      # for host in options.metastore_hosts
      #   @ambari.hosts.component_add
      #     header: 'HIVE_METASTORE'
      #     if: options.post_component
      #     url: options.ambari_url
      #     username: 'admin'
      #     password: options.ambari_admin_password
      #     cluster_name: options.cluster_name
      #     component_name: 'HIVE_METASTORE'
      #     hostname: host

      for host in options.hcatalog_hosts
        @ambari.hosts.component_add
          header: 'HIVE_METASTORE'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_METASTORE'
          hostname: host
        @ambari.hosts.component_add
          header: 'HIVE_METASTORE FIX'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_CLIENT'
          hostname: host

      for host in options.hcatalog_hosts
        @ambari.hosts.component_add
          header: 'HIVE_CLIENT'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_CLIENT'
          hostname: host

      # for host in [options.hcatalog_hosts]
      #   @ambari.hosts.component_add
      #     header: 'HIVE_CLIENT'
      #     if: options.post_component
      #     url: options.ambari_url
      #     username: 'admin'
      #     password: options.ambari_admin_password
      #     cluster_name: options.cluster_name
      #     component_name: 'HIVE_CLIENT'
      #     hostname: host


      for host in options.webhcat_hosts
        @ambari.hosts.component_add
          header: 'WEBHCAT_SERVER'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'WEBHCAT_SERVER'
          hostname: host

## Dependencies

    path = require 'path'
    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
