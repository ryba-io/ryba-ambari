
# Tez Install

    module.exports = header: 'Tez Install', handler: ({options}) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register 'hdfs_upload', 'ryba/lib/hdfs_upload'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"


## Render Configuration

      # @hconfigure
      #   header: 'Render tez-site'
      #   if: options.post_component and options.takeover
      #   source: "#{__dirname}/resources/tez-site.xml"
      #   target: "#{options.cache_dir}/tez-site.xml"
      #   ssh: false
      #   properties: options.tez_site
        
      @file.render
        header: 'Render tez-env'
        if: options.post_component and options.takeover
        source: "#{__dirname}/resources/tez-env.sh.j2"
        target: "#{options.cache_dir}/tez-env.sh"
        local: true
        ssh: false
        context:
          TEZ_CONF_DIR: options.env['TEZ_CONF_DIR']
          java_home: options.java_home
          hadoop_home: '/usr/hdp/current/hadoop-client'
        eof: true
        backup: true
        mode: 0o0750
        ssh: false


## Upload Configurations

Environment passed to Hadoop.

      # @call
      #   header: 'Upload tez-site'
      #   if: options.post_component and options.takeover
      # , (_, callback) ->
      #     properties.read null, "#{options.cache_dir}/tez-site.xml", (err, props) =>
      #       options.configurations['tez-site'] = props
      #       callback()

      @call
        header: 'Upload tez-env'
        if: options.post_component and options.takeover
      , (_, callback) ->
          ssh2fs.readFile null, "#{options.cache_dir}/tez-env.sh", (err, content) =>
            try
              throw err if err
              content = content.toString()
              options.configurations['tez-env'] = merge {}, options.configurations['tez-env'] , content: content
              callback()
            catch err
              callback err

## Upload Default Configuration

      @ambari.configs.default
        header: 'TEZ Configuration'
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        discover: true
        configurations: options.configurations
        target_services: 'TEZ'


## Add TEZ Service

      @ambari.services.add
        header: 'TEZ Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'TEZ'

      @ambari.services.wait
        header: 'TEZ Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'TEZ'

      @ambari.services.component_add
        header: 'TEZ_CLIENT Add'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'TEZ_CLIENT'
        service_name: 'TEZ'

## Install Component

      @ambari.hosts.component_add
        header: 'TEZ_CLIENT Host Add'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'TEZ_CLIENT'
        hostname: options.fqdn

      @ambari.hosts.component_wait
        header: 'TEZ_CLIENT Wait'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'TEZ_CLIENT'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'TEZ_CLIENT Install'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'TEZ_CLIENT'
        hostname: options.fqdn
        debug: true

## HDFS Tarballs

Upload the Tez tarball inside the "/hdp/apps/$version/tez"
HDFS directory. Note, the parent directories are created by the 
"ryba/hadoop/hdfs_dn/layout" module.

      @hdfs_upload
        header: 'HDFS Layout'
        source: '/usr/hdp/current/tez-client/lib/tez.tar.gz'
        target: '/hdp/apps/$version/tez/tez.tar.gz'
        lock: '/tmp/ryba-tez.lock'
        krb5_user: options.hdfs_krb5_user

# ## Tez UI
# 
# Tez UI will be untared in the tez.ui.html_path directory. A WebServer must be configured
# to serve this directory.
# 
#       @call header: 'UI', if: options.ui.enabled, ->
#         @system.mkdir
#           header: 'Layout'
#           target: options.ui.html_path
#         @system.execute
#           header: 'Web Files'
#           cmd: """
#           target_file=`ls /usr/hdp/current/tez-client/ui/tez-ui*.war | sed 's/^.*tez/tez/g'`
#           cd #{options.ui.html_path}
#           ls ${target_file} >/dev/null 2>&1
#           if [ $? -ne 0 ]; then
#             rm -rf *
#             cp /usr/hdp/current/tez-client/ui/tez-ui*.war .
#             jar xf tez-ui*.war
#           else
#             exit 3
#           fi
#           """
#           code_skipped: 3
#         @file
#           header: 'Env'
#           target: "#{options.ui.html_path}/config/configs.env"
#           content: "ENV = #{JSON.stringify options.ui.env, null, '  '};"
#           backup: true
#           eof: true
#         @file
#           header: 'Fix HTTPS'
#           target: "#{options.ui.html_path}/assets/tez-ui.js"
#           write: [
#             match: "      url = this.correctProtocol(url);"
#             replace: "      //url = this.correctProtocol(url);"
#           ]
#           backup: true

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
