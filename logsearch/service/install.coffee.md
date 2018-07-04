
# Ambari Logsearch Install

    module.exports =  header: 'Ambari Logsearch Install', handler: (options) ->
    
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','stacks','default'], 'ryba-ambari-actions/lib/stacks/default_informations'
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

          
      @call
        if: options.post_component and options.takeover
      , ->
        @call (opts, cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/zookeeper-logsearch-conf.json.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload zookeeper-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'zookeeper-logsearch-conf'
                cluster_name: options.cluster_name
                properties: 
                  'name': 'Zookeeper'
                  'component_mappings': 'ZOOKEEPER_SERVER:zookeeper'
                  content: content
              @next cb
            catch err
              callback err
        @call (opts, cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/hdfs-logsearch-conf.json.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload hdfs-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'hdfs-logsearch-conf'
                cluster_name: options.cluster_name
                properties:
                  service_name: 'HDFS'
                  'component_mappings': 'NAMENODE:hdfs_namenode;DATANODE:hdfs_datanode;SECONDARY_NAMENODE:hdfs_secondarynamenode;JOURNALNODE:hdfs_journalnode;ZKFC:hdfs_zkfc;NFS_GATEWAY:hdfs_nfs3'
                  content: content
              @next cb
            catch err
              cb err
        @call (opts,cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/mapred-logsearch-conf.json.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload mapred-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'mapred-logsearch-conf'
                cluster_name: options.cluster_name
                properties: 
                  service_name: 'MAPREDUCE2'
                  'component_mappings': 'HISTORYSERVER:mapred_historyserver'
                  content: content
              @next cb
            catch err
              cb err
        @call (opts,cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/yarn-logsearch-conf.json.j2", (err, content) =>
          
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload yarn-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'yarn-logsearch-conf'
                cluster_name: options.cluster_name
                properties:
                  'service_name': 'YARN'
                  'component_mappings': 'RESOURCEMANAGER:yarn_resourcemanager,yarn_historyserver,yarn_jobsummary;NODEMANAGER:yarn_nodemanager;APP_TIMELINE_SERVER:yarn_timelineserver'
                  content: content
              @next cb
            catch err
              cb err
          
        @call (opts,cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/atlas-logsearch-conf.json.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload atlas-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'atlas-logsearch-conf'
                cluster_name: options.cluster_name
                properties:
                  'service_name': 'ATLAS'
                  'component_mappings': 'ATLAS_SERVER:atlas_app'
                  content: content
              @next cb
            catch err
              cb err
        
        @call (opts,cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/hbase-logsearch-conf.json.j2", (err, content) =>
          
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload hbase-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'hbase-logsearch-conf'
                cluster_name: options.cluster_name
                properties: 
                  'service_name': 'HBASE'
                  'component_mappings': 'HBASE_MASTER:hbase_master;HBASE_REGIONSERVER:hbase_regionserver;PHOENIX_QUERY_SERVER:hbase_phoenix_server'
                  content: content
              @next cb
            catch err
              cb err

        @call (opts,cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/kafka-logsearch-conf.json.j2", (err, content) =>
          
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload kafka-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'kafka-logsearch-conf'
                cluster_name: options.cluster_name
                properties: 
                  'service_name': 'KAFKA'
                  'component_mappings': 'KAFKA_BROKER:kafka_server,kafka_request,kafka_logcleaner,kafka_controller,kafka_statechange'
                  content: content
              @next cb
            catch err
              cb err

        @call (opts,cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/ranger-logsearch-conf.json.j2", (err, content) =>
          
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload ranger-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'ranger-logsearch-conf'
                cluster_name: options.cluster_name
                properties: 
                  'service_name': 'RANGER'
                  'component_mappings': 'RANGER_SERVER:ranger_admin,ranger_dbpatch;RANGER_USERSYNC:ranger_usersync;'
                  content: content
              @next cb
            catch err
              cb err

        @call (opts,cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/spark-logsearch-conf.json.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload spark-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'spark-logsearch-conf'
                cluster_name: options.cluster_name
                properties: 
                  'service_name': 'SPARK'
                  'component_mappings': 'SPARK_JOBHISTORYSERVER:spark_jobhistory_server;SPARK_THRIFTSERVER:spark_thriftserver;LIVY_SERVER:livy_server'
                  content: content
              @next cb
            catch err
              cb err

        @call (opts,cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/spark2-logsearch-conf.json.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload spark2-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'spark2-logsearch-conf'
                cluster_name: options.cluster_name
                properties: 
                  'service_name': 'SPARK2'
                  'component_mappings': 'SPARK2_JOBHISTORYSERVER:spark2_jobhistory_server;SPARK2_THRIFTSERVER:spark2_thriftserver;LIVY2_SERVER:livy2_server'
                  content: content
              @next cb
            catch err
              cb err

        @call (opts,cb) ->
          ssh2fs.readFile null, "#{__dirname}/../resources/zeppelin-logsearch-conf.json.j2", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload spark2-logsearch-conf'
                if : options.post_component and options.takeover
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'zeppelin-logsearch-conf'
                cluster_name: options.cluster_name
                properties: 
                  'service_name': 'ZEPPELIN'
                  'component_mappings': 'ZEPPELIN_MASTER:zeppelin'
                  content: content
              @next cb
            catch err
              cb err

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