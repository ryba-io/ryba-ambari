
# Hadoop HDFS Client Install

    module.exports = header: 'HDFS Client Ambari Install', handler: ({options}) ->

## Registry

      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## Hadoop HDFS Site

Update the "hdfs-site.xml" configuration file with properties from the
"ryba.hdfs.site" configuration.

      @call header: 'Jars', ->
        core_jars = Object.keys(options.core_jars).map (k) -> options.core_jars[k]
        remote_files = null
        @call (_, callback) ->
          ssh = @ssh options.ssh
          ssh2fs.readdir ssh, '/usr/hdp/current/hadoop-hdfs-client/lib', (err, files) ->
            remote_files = files unless err
            callback err
        @call ->
          remove_files = []
          core_jars = for jar in core_jars
            filtered_files = multimatch remote_files, jar.match
            remove_files.push (filtered_files.filter (file) -> file isnt jar.filename)...
            continue if jar.filename in remote_files
            jar
          @system.remove ( # Remove jar if already uploaded
            target: path.join '/usr/hdp/current/hadoop-hdfs-client/lib', file
          ) for file in remove_files
          @file.download (
            source: jar.source
            target: path.join '/usr/hdp/current/hadoop-hdfs-client/lib', "#{jar.filename}"
          ) for jar in core_jars
          @file.download (
            source: jar.source
            target: path.join '/usr/hdp/current/hadoop-yarn-client/lib', "#{jar.filename}"
          ) for jar in core_jars


### HDFS_CLIENT component wait
Wait for the HDFS_CLIENT component to be declared on the host

      @ambari.hosts.component_wait
        header: 'HDFS_CLIENT WAITED'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HDFS_CLIENT'
        hostname: options.fqdn

### HDFS_CLIENT component install
Put the HDFS_CLIENT component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'HDFS_CLIENT set installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HDFS_CLIENT'
        hostname: options.fqdn

## Dependencies

    ssh2fs = require 'ssh2-fs'
