
# Ambari Logsearch Server Stop

Stop the LOGSEARCH_SERVER service through ambari.

    module.exports = header: 'Ambari Logsearch Server Stop', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Service Stop

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'LOGSEARCH_SERVER'
        hostname: options.fqdn

