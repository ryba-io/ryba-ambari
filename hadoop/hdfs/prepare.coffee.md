
# Hadoop Takeover Prepare
Prepare scripts and files before taking over the cluster.
For example the hadoop env file is rendered with all variable.

    module.exports = header: 'Hadoop Takeover', handler: (options) ->
      return unless options.post_component
      
## Registry

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'

## Render Files

      @call
        header: 'Hadoop Env'
      , ->
          HADOOP_NAMENODE_OPTS = options.hdfs_nn_opts.base
          HADOOP_NAMENODE_OPTS += " -D#{k}=#{v}" for k, v of options.hdfs_nn_opts.java_properties
          HADOOP_NAMENODE_OPTS += " #{k}#{v}" for k, v of options.hdfs_nn_opts.jvm
          HADOOP_DATANODE_OPTS = options.hdfs_dn_opts.base
          HADOOP_DATANODE_OPTS += " -D#{k}=#{v}" for k, v of options.hdfs_dn_opts.java_properties
          HADOOP_DATANODE_OPTS += " #{k}#{v}" for k, v of options.hdfs_dn_opts.jvm
          HADOOP_JOURNALNODE_OPTS = options.hdfs_jn_opts.base
          HADOOP_JOURNALNODE_OPTS += " -D#{k}=#{v}" for k, v of options.hdfs_jn_opts.java_properties
          HADOOP_JOURNALNODE_OPTS += " #{k}#{v}" for k, v of options.hdfs_jn_opts.jvm
          @file.render
            header: 'Render'
            source: "#{__dirname}/../resources/hadoop-env.sh.j2"
            target: "#{options.cache_dir}/hadoop-env.sh"
            ssh: false
            context: merge options.configurations['hadoop-env'],
              HADOOP_NAMENODE_OPTS: HADOOP_NAMENODE_OPTS
              HADOOP_DATANODE_OPTS: HADOOP_DATANODE_OPTS
              HADOOP_JOURNALNODE_OPTS: HADOOP_JOURNALNODE_OPTS

      @call
        header: 'Yarn Env'
      , ->
          YARN_RESOURCEMANAGER_OPTS = options.yarn_rm_opts.base
          YARN_RESOURCEMANAGER_OPTS += " -D#{k}=#{v}" for k, v of options.yarn_rm_opts.java_properties
          YARN_RESOURCEMANAGER_OPTS += " #{k}#{v}" for k, v of options.yarn_rm_opts.jvm
          YARN_NODEMANAGER_OPTS = options.yarn_nm_opts.base
          YARN_NODEMANAGER_OPTS += " -D#{k}=#{v}" for k, v of options.yarn_nm_opts.java_properties
          YARN_NODEMANAGER_OPTS += " #{k}#{v}" for k, v of options.yarn_nm_opts.jvm
          YARN_TIMELINESERVER_OPTS = options.yarn_ts_opts.base
          YARN_TIMELINESERVER_OPTS += " -D#{k}=#{v}" for k, v of options.yarn_ts_opts.java_properties
          YARN_TIMELINESERVER_OPTS += " #{k}#{v}" for k, v of options.yarn_ts_opts.jvm
          @file.render
            header: 'Render'
            source: "#{__dirname}/../resources/yarn-env.sh.j2"
            target: "#{options.cache_dir}/yarn-env.sh"
            ssh: false
            context: merge options.configurations['yarn-env'],
              YARN_RESOURCEMANAGER_OPTS: YARN_RESOURCEMANAGER_OPTS
              YARN_NODEMANAGER_OPTS: YARN_NODEMANAGER_OPTS
              YARN_HISTORYSERVER_OPTS: YARN_TIMELINESERVER_OPTS
              YARN_TIMELINESERVER_OPTS: YARN_TIMELINESERVER_OPTS

## Log4j

      @file
        header: 'HDFS Log4j'
        target: "#{options.cache_dir}/hdfs-log4j.properties"
        source: "#{__dirname}/../resources/log4j.properties"
        ssh: false
        write: for k, v of options.hdfs_log4j.properties
          match: RegExp "#{k}=.*", 'm'
          replace: "#{k}=#{v}"
          append: true

      @file
        header: 'YARN Log4j'
        target: "#{options.cache_dir}/yarn-log4j.properties"
        source: "#{__dirname}/../resources/log4j.properties"
        ssh: false
        write: for k, v of options.yarn_log4j.properties
          match: RegExp "#{k}=.*", 'm'
          replace: "#{k}=#{v}"
          append: true
        
## Site

      @hconfigure
        header: 'Render Core Site'
        target: "#{options.cache_dir}/core-site.xml"
        source: "#{__dirname}/../../resources/core_hadoop/core-site.xml"
        local: true
        properties: options.configurations['core-site']
        ssh: false
        backup: true
      @hconfigure
        header: 'HDFS Site'
        target: "#{options.cache_dir}/hdfs-site.xml"
        source: "#{__dirname}/../../resources/core_hadoop/hdfs-site.xml"
        ssh: false
        properties: options.configurations['hdfs-site']
        local: true
        backup: false
      @hconfigure
        header: 'Yarn Site'
        target: "#{options.cache_dir}/yarn-site.xml"
        source: "#{__dirname}/../../resources/core_hadoop/yarn-site.xml"
        local: true
        ssh: false
        properties: options.configurations['yarn-site']
        backup: true
      @hconfigure
        header: 'Mapred Site'
        target: "#{options.cache_dir}/mapred-site.xml"
        source: "#{__dirname}/../../resources/core_hadoop/mapred-site.xml"
        ssh: false
        local: true
        properties: options.configurations['mapred-site']
        backup: true
      @hconfigure
        target: "#{options.cache_dir}/ssl-server.xml"
        properties: options.configurations['ssl-server']
        ssh: false
      @hconfigure
        target: "#{options.cache_dir}/ssl-client.xml"
        ssh: false
        properties: options.configurations['ssl-client']

## Upload Configs

## HDFS-SITE, YARN-SITE, MAPRED-SITE
Update hdfs-site.xml, yarn-site.xml, mapred-site.xml

      @ambari.configs.update
        header: 'Upload HDFS Site'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hdfs-site'
        cluster_name: options.cluster_name
        properties: {}
        debug: true
        tag: 'version4'
        version: 4

      @ambari.configs.update
        header: 'Upload Yarn Site'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'yarn-site'
        cluster_name: options.cluster_name
        properties: options.configurations['yarn-site']

      @ambari.configs.update
        header: 'Upload Mapred Site'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'mapred-site'
        cluster_name: options.cluster_name
        properties: options.configurations['mapred-site']

## Hadoop Policy

      @ambari.configs.update
        header: 'Hadoop Policy'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hadoop-policy'
        cluster_name: options.cluster_name
        properties: options.hadoop_policy

## Hadoop Log4j

      @ambari.configs.update
        header: 'HDFS Log4j'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'hdfs-log4j'
        cluster_name: options.cluster_name
        properties: options.hdfs_log4j
    
      @ambari.configs.update
        header: 'YARN Log4j'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'yarn-log4j'
        cluster_name: options.cluster_name
        properties: options.yarn_log4j


## HADOOP-ENV, YARN-ENV
Render hadoop-env.sh and yarn-env.sh files, before uploading to Ambari Server.

      @call
        header: 'Hadoop Env'
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/hadoop-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Update to Ambari'
                url: options.ambari_url
                username: 'admin'
                merge: true
                password: options.ambari_admin_password
                config_type: 'hadoop-env'
                cluster_name: options.cluster_name
                properties: merge {},
                  hadoop_pid_dir_prefix: options.configurations['hadoop-env'].hadoop_pid_dir_prefix
                  hdfs_log_dir_prefix: options.configurations['hadoop-env'].hdfs_log_dir_prefix
                  hadoop_root_logger: options.configurations['hadoop-env'].hadoop_root_logger
                  hadoop_heapsize: options.configurations['hadoop-env'].hadoop_heapsize
                  namenode_heapsize: options.configurations['hadoop-env'].namenode_heapsize
                  namenode_opt_newsize: options.configurations['hadoop-env'].namenode_opt_newsize
                  namenode_opt_maxnewsize: options.configurations['hadoop-env'].namenode_opt_maxnewsize
                  namenode_opt_permsize: '128m'
                  namenode_opt_maxpermsize: '256m'
                  hdfs_user: options.configurations['hadoop-env'].hdfs_user
                  hdfs_user_keytab: options.configurations['hadoop-env'].hdfs_user_keytab
                  hdfs_principal_name: options.configurations['hadoop-env'].hdfs_principal_name
                  hdfs_tmp_dir: options.configurations['hadoop-env'].hdfs_tmp_dir
                  hadoop_root_logger: options.configurations['hadoop-env'].hadoop_root_logger
                  hdfs_user_nofile_limit:  options.configurations['hadoop-env'].hdfs_user_nofile_limit
                  hdfs_user_nproc_limit:  options.configurations['hadoop-env'].hdfs_user_nproc_limit
                ,  
                  content: content
                  
                  # zk_log_dir: options.log_dir
                  # zk_pid_dir: '/var/run/zookeeper'
                  # zk_user: options.user.name
                  # tickTime: options.config['tickTime']
                  # initLimit: options.config['initLimit']
                  # syncLimit: options.config['syncLimit']
                  # clientPort: options.config['clientPort']
                  # zookeeper_keytab_path: options.krb5.keytab
                  # zookeeper_principal_name: options.krb5.principal
              .next callback
            catch err
              callback err

      @call
        header: 'Yarn Env'
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/yarn-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Update to Ambari'
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'yarn-env'
                cluster_name: options.cluster_name
                properties: 
                  content: content
                  yarn_user: options.configurations['yarn-env']['yarn_user']
                  yarn_tmp_dir: options.configurations['yarn-env']['yarn_tmp_dir']
                  yarn_user_nofile_limit: options.configurations['yarn-env']['yarn_user_nofile_limit']
                  yarn_user_nproc_limit: options.configurations['yarn-env']['yarn_user_nproc_limit']
                  yarn_heapsize: options.configurations['yarn-env']['yarn_heapsize']
                  nodemanager_heapsize: options.configurations['yarn-env']['nodemanager_heapsize']
                  resourcemanager_heapsize: options.configurations['yarn-env']['resourcemanager_heapsize']
                  apptimelineserver_heapsize: options.configurations['yarn-env']['apptimelineserver_heapsize']
                  hadoop_yarn_home: options.configurations['yarn-env']['hadoop_yarn_home']
                  yarn_log_dir_prefix: options.configurations['yarn-env']['yarn_log_dir_prefix']
                  hadoop_libexec_dir: options.configurations['yarn-env']['hadoop_libexec_dir']
                  yarn_pid_dir_prefix: options.configurations['yarn-env']['yarn_pid_dir_prefix']
              .next callback
            catch err
              callback err

## Log4j

      @call
        header: 'HDFS Log4j'
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/hdfs-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Update To ambari'
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'hdfs-log4j'
                cluster_name: options.cluster_name
                properties: 
                  content: content
              .next callback
            catch err
              callback err

      @call
        header: 'YARN Log4j'
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/yarn-log4j.properties", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'config update'
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'yarn-log4j'
                cluster_name: options.cluster_name
                properties: 
                  content: content
                  # zk_log_dir: options.log_dir
                  # zk_pid_dir: '/var/run/zookeeper'
                  # zk_user: options.user.name
                  # tickTime: options.config['tickTime']
                  # initLimit: options.config['initLimit']
                  # syncLimit: options.config['syncLimit']
                  # clientPort: options.config['clientPort']
                  # zookeeper_keytab_path: options.krb5.keytab
                  # zookeeper_principal_name: options.krb5.principal
              .next callback
            catch err
              callback err

      @call
        header: 'Core site'
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/core-site.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update core-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'core-site'
              cluster_name: options.cluster_name
              properties: props
            @next callback

      @call
        header: 'HDFS site'
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/hdfs-site.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update hdfs-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'hdfs-site'
              cluster_name: options.cluster_name
              properties: props
            @next callback

      @call
        header: 'Yarn site'
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/yarn-site.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update yarn-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'yarn-site'
              cluster_name: options.cluster_name
              properties: props
            @next callback

      @call
        header: 'Mapred site'
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/mapred-site.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update mapred-site'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'mapred-site'
              cluster_name: options.cluster_name
              properties: props
            @next callback

      @call
        header: 'SSl Server'
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/ssl-server.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'ssl-server'
              cluster_name: options.cluster_name
              properties: props
            @next callback

      @call
        header: 'SSl Client'
      , (_, callback) ->
          properties.read null, "#{options.cache_dir}/ssl-client.xml", (err, props) =>
            @ambari.configs.update
              header: 'config update'
              url: options.ambari_url
              username: 'admin'
              password: options.ambari_admin_password
              config_type: 'ssl-client'
              cluster_name: options.cluster_name
              properties: props
            @next callback
      
      @ambari.configs.update
        header: 'Scheduler to ambari'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        config_type: 'capacity-scheduler'
        cluster_name: options.cluster_name
        properties: options.configurations['capacity-scheduler']


## Metrics Properties
  
      @file.properties
        if: options.ambari_host
        header: 'Metrics Render'
        target: "#{options.cache_dir}/hadoop-metrics2.properties"
        ssh: false
        content: options.configurations['hadoop-metrics-properties']

      @file.download
        if: options.ambari_host
        header: 'Metrics Upload'
        target: '/var/lib/ambari-server/resources/stacks/HDP/2.0.6/hooks/before-START/templates/hadoop-metrics2.properties.j2'
        source: "#{options.cache_dir}/hadoop-metrics2.properties"
        backup: true
        ssh: true

## Dependencies

    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require '../../lib/properties'
