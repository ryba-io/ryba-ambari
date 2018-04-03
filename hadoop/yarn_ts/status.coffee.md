
# Hadoop YARN Timeline Server Start

## Status

Check if the Timeline Server is running. The process ID is located by default
inside "/var/run/hadoop-yarn/yarn-yarn-timelineserver.pid" (TODO, check the pid file!).

    module.exports = header: 'YARN ATS Ambari Status', handler: ->

## Registry

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'

## Service Status

      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'APP_TIMELINE_SERVER'
        hostname: options.fqdn
