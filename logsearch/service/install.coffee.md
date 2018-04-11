
# Ambari Logsearch Install

    module.exports =  header: 'Ambari Logsearch Install', handler: (options) ->
    
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

## Upload Solr 7.+

## service_logs-solrconfig
Render hadoop-env.sh and yarn-env.sh files, before uploading to Ambari Server.

      @call
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/service_logs-solrconfig-#{options.download}.xml.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations = merge {}, options.configurations,
                'logsearch-service_logs-solrconfig':
                  'content': content
              callback()
            catch err
              callback err

      @call
        if: options.post_component
      , (_, callback) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/audit_logs-solrconfig-#{options.download}.xml.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations = merge {}, options.configurations,
                'logsearch-audit_logs-solrconfig':
                  'content': content
              callback()
            catch err
              callback err


      # @file.download
      #   local: true
      #   source: "#{__dirname}/../resources/service_logs-solrconfig.xml.j2"
      #   target: "/var/lib/ambari-server/resources/common-services/LOGSEARCH/0.5.0/properties/service_logs-solrconfig.xml.j2"
      #   backup: true
      # @file.download
      #   local: true
      #   source: "#{__dirname}/../resources/audit_logs-solrconfig.xml.j2"
      #   target: "/var/lib/ambari-server/resources/common-services/LOGSEARCH/0.5.0/properties/audit_logs-solrconfig.xml.j2"
      #   backup: true
      # @file.download
      #   local: true
      #   source: "#{__dirname}/../resources/service_logs-solrconfig.xml.j2"
      #   target: "/var/lib/ambari-agent/cache/common-services/LOGSEARCH/0.5.0/properties/service_logs-solrconfig.xml.j2"
      #   backup: true
      # @file.download
      #   local: true
      #   source: "#{__dirname}/../resources/audit_logs-solrconfig.xml.j2"
      #   target: "/var/lib/ambari-agent/cache/common-services/LOGSEARCH/0.5.0/properties/audit_logs-solrconfig.xml.j2"
      #   backup: true


## Upload Default Configuration

      @call ->
        @ambari.configs.default
          header: 'LOGSEARCH Configuration'
          url: options.ambari_url
          if: options.post_component and options.takeover
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          stack_name: options.stack_name
          stack_version: options.stack_version
          discover: true
          configurations: options.configurations
          target_services: 'LOGSEARCH'

## Add LOGSEARCH Service

      @ambari.services.add
        header: 'LOGSEARCH Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'LOGSEARCH'

      @ambari.services.wait
        header: 'LOGSEARCH Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'LOGSEARCH'

      @ambari.services.component_add
        header: 'LOGSEARCH_SERVER Add'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'LOGSEARCH_SERVER'
        service_name: 'LOGSEARCH'

      @ambari.services.component_add
        header: 'LOGSEARCH_LOGFEEDER Add'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'LOGSEARCH_LOGFEEDER'
        service_name: 'LOGSEARCH'

## Install Component

      for host in options.feeder_hosts
        @ambari.hosts.component_add
          header: 'LOGSEARCH_LOGFEEDER Host Add'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'LOGSEARCH_LOGFEEDER'
          hostname: host

      for host in options.server_hosts
        @ambari.hosts.component_add
          header: 'LOGSEARCH_SERVER Host Add'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'LOGSEARCH_SERVER'
          hostname: host

## Dependencies

    ssh2fs = require 'ssh2-fs'
    {merge} = require 'nikita/lib/misc'