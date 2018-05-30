
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
      
## Layout

Create the directories to store the logs and pid information. The properties
"ryba.hive.server2.log\_dir" and "ryba.hive.server2.pid\_dir" may be modified.

      @call header: 'Layout', ->
        @system.mkdir
          target: options.log_dir
          uid: options.user.name
          gid: options.group.name
          parent: true
        @system.mkdir
          target: options.pid_dir
          uid: options.user.name
          gid: options.group.name
          parent: true



## Render Configuration

      @hconfigure
        header: 'Render hive-site'
        if: options.post_component and options.takeover
        source: "#{__dirname}/../resources/hive-site.xml"
        target: "#{options.cache_dir}/hive-site.xml"
        ssh: false
        properties: options.configurations['hive-site']

      @file.render
        header: 'Render hive-exec-log4j2'
        if: options.post_component and options.takeover
        source: "#{__dirname}/../resources/hive-exec-log4j.properties.j2"
        local: true
        target: "#{options.cache_dir}/hive-exec-log4j.properties"
        context: options
        ssh: false
      @file.properties
        header: 'Render hive-log4j2'
        if: options.post_component and options.takeover
        target: "#{options.cache_dir}/hive-log4j.properties"
        content: options.hive_log4j
        backup: true
        ssh: false

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
      @file
        header: 'Render webhcat-log4j'
        if: options.post_component and options.webhcat_log4j? and options.takeover
        target: "#{options.cache_dir}/webhcat-log4j.properties"
        source: "#{__dirname}/../resources/webhcat-log4j.properties"
        local: true
        ssh: false
        write: for k, v of options.webhcat_log4j
          match: RegExp "#{k}=.*", 'm'
          replace: "#{k}=#{v}"
          append: true
      @hconfigure
        header: 'Render webhcat-site'
        if: options.post_component and options.takeover
        source: "#{__dirname}/../resources/webhcat-site.xml"
        target: "#{options.cache_dir}/webhcat-site.xml"
        ssh: false
        properties: options.configurations['webhcat-site']

## Upload Configurations
Upload hive-env, hive-site, hive-exec-log4j2, hive-log4j2, webhcat-env, webhcat-site
and webhcat-log4j

      # @call
      #   header: 'Upload hive-env'
      #   if: options.post_component
      # , (_, callback) ->
      #     ssh2fs.readFile null, "#{options.cache_dir}/hive-env.sh", (err, content) =>
      #       try
      #         throw err if err
      #         content = content.toString()
      #         @ambari.configs.update
      #           url: options.ambari_url
      #           username: 'admin'
      #           merge: true
      #           password: options.ambari_admin_password
      #           config_type: 'hive-env'
      #           cluster_name: options.cluster_name
      #           properties: merge {},  options.configurations['hive-env'],
      #             content: content
      #         .next callback
      #       catch err
      #         callback err
      # 
      # @call
      #   header: 'Upload hive-interactive-env'
      #   if: options.post_component
      # , (_, callback) ->
      #     ssh2fs.readFile null, "#{options.cache_dir}/hive-interactive-env.sh", (err, content) =>
      #       try
      #         throw err if err
      #         content = content.toString()
      #         @ambari.configs.update
      #           url: options.ambari_url
      #           username: 'admin'
      #           merge: true
      #           password: options.ambari_admin_password
      #           config_type: 'hive-interactive-env'
      #           cluster_name: options.cluster_name
      #           properties: merge {},  options.configurations['hive-interactive-env'],
      #             content: content
      #         .next callback
      #       catch err
      #         callback err

      @call
        header: 'Upload hcat-env'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/hcat-env.sh.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hcat-env'
                cluster_name: options.cluster_name
                properties: merge {},  options.configurations['hcat-env'],
                  content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload hive-env'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/hive-env.sh.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hive-env'
                cluster_name: options.cluster_name
                properties: merge {},  options.configurations['hive-env'],
                  content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload hive-interactive-env'
        if: options.post_component and options.takeover
      , (_, callback) ->
          console.log 'TODO: CHECK|ING add merge hive-interactive-env'
          ssh2fs.readFile null, "#{__dirname}/../resources/hive-env.sh.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hive-interactive-env'
                cluster_name: options.cluster_name
                properties: merge {},  options.configurations['hive-interactive-env']
                  # content: content
              .next callback
            catch err
              callback err



      @call
        header: 'Upload hive-site'
        if: options.post_component and options.takeover
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/hive-site.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update hive-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'hive-site'
              cluster_name: options.cluster_name
              properties: props
            @ambari.configs.update
              header: 'config update hive-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'hivemetastore-site'
              cluster_name: options.cluster_name
              properties: props
            @next callback

      @ambari.configs.update
        header: 'Upload hive-interactive-site'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hive-interactive-site'
        cluster_name: options.cluster_name
        properties: options.configurations['hive-interactive-site']

      @call
        header: 'Upload hive-exec-log4j2'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/hive-exec-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hive-exec-log4j2'
                cluster_name: options.cluster_name
                properties: content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload hive-log4j2'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/hive-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hive-log4j2'
                cluster_name: options.cluster_name
                properties: content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload webhcat-env'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/webhcat-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'webhcat-env'
                cluster_name: options.cluster_name
                properties: merge {},  options.configurations['webhcat-env'],
                  content: content
              .next callback
            catch err
              callback err

      @call
        header: 'Upload webhcat-site'
        if: options.post_component and options.takeover
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/webhcat-site.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update webhcat-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'webhcat-site'
              cluster_name: options.cluster_name
              properties: props
            @next callback


      @call
        header: 'Upload webhcat-log4j'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/webhcat-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'webhcat-log4j'
                cluster_name: options.cluster_name
                properties: content: content
              .next callback
            catch err
              callback err

      @ambari.configs.update
        header: 'Upload hiveserver2-site'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hiveserver2-site'
        cluster_name: options.cluster_name
        properties: options.configurations['hiveserver2-site']

## Upload Ranger Related Properties

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
        if: options.post_component and options.baremetal
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
        if: options.post_component and options.webhcat_log4j? and options.baremetal
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
        if: options.post_component and options.baremetal
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
        if: options.post_component and options.baremetal
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
        if: options.post_component and options.baremetal
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

## RANGER PLUGIN Properties

      @ambari.configs.update
        header: 'Upload ranger-hive-plugin-properties'
        if : options.post_component and options.baremetal
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-plugin-properties'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-plugin-properties']

      @ambari.configs.update
        header: 'Upload ranger-hive-security'
        if : options.post_component and options.baremetal
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-security'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-security']

      @ambari.configs.update
        header: 'Upload ranger-hive-policymgr-ssl'
        if : options.post_component and options.baremetal
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-policymgr-ssl'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-policymgr-ssl']

      @ambari.configs.update
        header: 'Upload ranger-hive-audit'
        if : options.post_component and options.baremetal
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'ranger-hive-audit'
        cluster_name: options.cluster_name
        properties: options.configurations['ranger-hive-audit']

## Add HIVE Service

      @ambari.services.add
        header: 'HIVE Service'
        if: options.post_component and (options.takeover or options.baremetal)
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
        if: options.post_component and (options.takeover or options.baremetal)
        header: 'HIVE_SERVER'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_SERVER'
        service_name: 'HIVE'

      @ambari.services.component_add
        if: options.post_component and (options.takeover or options.baremetal)
        header: 'HCAT'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HCAT'
        service_name: 'HIVE'
        
      @ambari.services.component_add
        if: options.post_component and (options.takeover or options.baremetal)
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
        if: options.post_component and (options.takeover or options.baremetal)
        header: 'HIVE_METASTORE'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HIVE_METASTORE'
        service_name: 'HIVE'

      @ambari.services.component_add
        if: options.post_component and (options.takeover or options.baremetal)
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
          if: options.post_component and (options.takeover or options.baremetal)
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
          if: options.post_component and (options.takeover or options.baremetal)
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_METASTORE'
          hostname: host
        @ambari.hosts.component_add
          header: 'HIVE_METASTORE FIX'
          if: options.post_component and (options.takeover or options.baremetal)
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_CLIENT'
          hostname: host

      for host in options.hcatalog_hosts
        @ambari.hosts.component_add
          header: 'HIVE_CLIENT'
          if: options.post_component and (options.takeover or options.baremetal)
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HIVE_CLIENT'
          hostname: host

      for host in options.webhcat_hosts
        @ambari.hosts.component_add
          header: 'WEBHCAT_SERVER'
          if: options.post_component and (options.takeover or options.baremetal)
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
