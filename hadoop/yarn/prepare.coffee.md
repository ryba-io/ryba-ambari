
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
        header: 'Yarn Env'
        if: options.post_component
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
            source: "#{__dirname}/../resources/yarn-env.sh.ambari.j2"
            target: "#{options.cache_dir}/yarn-env-prometheus.sh"
            ssh: false
            context: merge options.configurations['yarn-env'],
              YARN_RESOURCEMANAGER_OPTS: YARN_RESOURCEMANAGER_OPTS
              YARN_NODEMANAGER_OPTS: YARN_NODEMANAGER_OPTS
              YARN_HISTORYSERVER_OPTS: YARN_TIMELINESERVER_OPTS
              YARN_TIMELINESERVER_OPTS: YARN_TIMELINESERVER_OPTS

## Dependencies

    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
