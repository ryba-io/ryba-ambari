
# Ambari Metrics Monitor

Start the Metrics  Monitor service through ambari.

    module.exports = header: 'Ambari Metrics Monitor Start', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Start

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_MONITOR'
        hostname: options.fqdn

