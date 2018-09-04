
# Ambari Logsearch Server Start

Start the LOGSEARCH_SERVER service through ambari.

    module.exports = header: 'Ambari Logsearch Server Start', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Start

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'LOGSEARCH_SERVER'
        hostname: options.fqdn

