
# Ambari Takeover

    module.exports = header: 'HDFS Ambari Install', handler: (options) ->
      
## Register

        

      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','configs','get'], 'ryba-ambari-actions/lib/configs/get'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_update'], "ryba-ambari-actions/lib/hosts/component_update"
      @registry.register ['ambari','configs','groups_add'], 'ryba-ambari-actions/lib/configs/groups/add'
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'
      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'


## Render Files

      @call
        if: options.post_component
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
          HADOOP_ZKFC_OPTS = options.zkfc_opts.base
          HADOOP_ZKFC_OPTS += " -D#{k}=#{v}" for k, v of options.zkfc_opts.java_properties
          HADOOP_ZKFC_OPTS += " #{k}#{v}" for k, v of options.zkfc_opts.jvm
          @file.render
            header: 'Render'
            source: "#{__dirname}/../resources/hadoop-env.sh.ambari.j2"
            target: "#{options.cache_dir}/hadoop-env-prometheus.sh"
            ssh: false
            context:
              HADOOP_NAMENODE_OPTS: HADOOP_NAMENODE_OPTS
              HADOOP_DATANODE_OPTS: HADOOP_DATANODE_OPTS
              HADOOP_JOURNALNODE_OPTS: HADOOP_JOURNALNODE_OPTS
              HADOOP_ZKFC_OPTS: HADOOP_ZKFC_OPTS

## HADOOP-ENV
Render hadoop-env.sh and yarn-env.sh files, before uploading to Ambari Server.

      @call
        header: 'Hadoop Env'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/hadoop-env-prometheus.sh", (err, content) =>
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
                properties: merge {}, options.configurations['hadoop-env'], content: content.toString()
                # hadoop_pid_dir_prefix: options.configurations['hadoop-env'].hadoop_pid_dir_prefix
                # hdfs_log_dir_prefix: options.configurations['hadoop-env'].hdfs_log_dir_prefix
                # hadoop_root_logger: options.configurations['hadoop-env'].hadoop_root_logger
                # hadoop_heapsize: options.configurations['hadoop-env'].hadoop_heapsize
                # namenode_heapsize: options.configurations['hadoop-env'].namenode_heapsize
                # namenode_opt_newsize: options.configurations['hadoop-env'].namenode_opt_newsize
                # namenode_opt_maxnewsize: options.configurations['hadoop-env'].namenode_opt_maxnewsize
                # namenode_opt_permsize: '128m'
                # namenode_opt_maxpermsize: '256m'
                # hdfs_user: options.configurations['hadoop-env'].hdfs_user
                # hdfs_user_keytab: options.configurations['hadoop-env'].hdfs_user_keytab
                # hdfs_principal_name: options.configurations['hadoop-env'].hdfs_principal_name
                # hdfs_tmp_dir: options.configurations['hadoop-env'].hdfs_tmp_dir
                # hadoop_root_logger: options.configurations['hadoop-env'].hadoop_root_logger
                # hdfs_user_nofile_limit:  options.configurations['hadoop-env'].hdfs_user_nofile_limit
                # hdfs_user_nproc_limit:  options.configurations['hadoop-env'].hdfs_user_nproc_limit
                # hadoop_conf_secure_dir:  options.configurations['hadoop-env'].hadoop_conf_secure_dir
                # hadoop_conf_dir:  options.configurations['hadoop-env'].hadoop_conf_dir
                # proxyuser_group:  options.configurations['hadoop-env'].proxyuser_group
                # java_home:
                # dtnode_heapsize
              .next callback
            catch err
              callback err

## Dependencies

    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
