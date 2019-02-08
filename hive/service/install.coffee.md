
# Hive Service Install

    module.exports =  header: 'Ambari Hive Service Install', handler: ({options}) ->

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
          if: options.post_component and options.takeover
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
        if: options.post_component and options.webhcat_log4j? and options.takeover
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
        if: options.post_component and options.takeover
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
        if: options.post_component and options.takeover
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
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/hive-env.sh.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations['hive-env'] =  merge {},  options.configurations['hive-env'], content: content
              callback()
            catch err
              callback err

## Dependencies

    path = require 'path'
    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
