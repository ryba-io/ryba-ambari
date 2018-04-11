
# Ambari Takeover

    module.exports = header: 'Mapreduce Ambari Install', handler: (options) ->
      
## Register

      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'

## Packages

Install the "hadoop-client" and "openssl" packages as well as their
dependecies.

The environment script "hadoop-env.sh" from the HDP companion files is also
uploaded when the package is first installed or upgraded. Be careful, the
original file will be overwritten with and user modifications. A copy will be
made available in the same directory after any modification.

      @call header: 'Packages', ->
        @service
          name: 'openssl-devel'
        @service
          name: 'hadoop-client'
        @hdp_select
          name: 'hadoop-client'

## Upload Configs
Update mapred-env.sh


      @call
        header: 'mapred-env'
      , ->
      @file.render
        header: 'Render mapred-env'
        if: options.post_component
        source: "#{__dirname}/../resources/mapred-env.sh.j2"
        target: "#{options.cache_dir}/mapred-env.sh"
        ssh: false
        context: merge options.configurations['mapred-env'],
          HADOOP_MAPRED_LOG_DIR: options.configurations['mapred-env']['mapred_log_dir_prefix']
          HADOOP_MAPRED_PID_DIR: options.mapred.pid_dir


      @call
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/mapred-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              @ambari.configs.update
                header: 'Upload mapred-env'
                url: options.ambari_url
                username: 'admin'
                password: options.ambari_admin_password
                config_type: 'mapred-env'
                cluster_name: options.cluster_name
                properties: 
                  content: content
                  mapred_user: options.configurations['mapred-env']['mapred_user']
                  mapred_tmp_dir: options.configurations['mapred-env']['mapred_tmp_dir']
                  mapred_user_nofile_limit: options.configurations['mapred-env']['mapred_user_nofile_limit']
                  mapred_user_nproc_limit: options.configurations['mapred-env']['mapred_user_nproc_limit']
                  jobhistory_heapsize: options.configurations['mapred-env']['jobhistory_heapsize']
                  mapred_pid_dir_prefix: options.configurations['mapred-env']['mapred_pid_dir_prefix']
                  mapred_log_dir_prefix: options.configurations['mapred-env']['mapred_log_dir_prefix']
                  mapred_jobstatus_dir: options.configurations['mapred-env']['mapred_jobstatus_dir']
              .next callback
            catch err
              callback err

### MAPREDUCE Service
      
      @ambari.services.add
        header: 'MAPREDUCE2 Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'MAPREDUCE2'

      @ambari.services.wait
        header: 'MAPREDUCE2 Wait'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'MAPREDUCE2'
        
      @ambari.services.component_add
        header: 'HISTORYSERVER'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HISTORYSERVER'
        service_name: 'MAPREDUCE2'
        
      @ambari.services.component_add
        header: 'MAPREDUCE2_CLIENT'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'MAPREDUCE2_CLIENT'
        service_name: 'MAPREDUCE2'

      for host in options.mapred_jhs_hosts
        @ambari.hosts.component_add
          header: 'HISTORYSERVER ADD'
          url: options.ambari_url
          if: options.takeover
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'HISTORYSERVER'
          hostname: host

      for host in options.client_hosts
        @ambari.hosts.component_add
          header: 'MAPREDUCE2_CLIENT ADD'
          url: options.ambari_url
          if: options.takeover
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'MAPREDUCE2_CLIENT'
          hostname: host

## Dependencies

    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
