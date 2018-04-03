
# YARN NodeManager Status

Check if the Yarn NodeManager server is running. The process ID is located by
default inside "/var/run/hadoop-yarn/yarn-yarn-nodemanager.pid".

    module.exports = header: 'YARN NM Ambari Status', handler: ->

## Registry

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'

## Service Status

      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'NODEMANAGER'
        hostname: options.fqdn

