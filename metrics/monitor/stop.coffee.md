
# Ambari Metrics Monitor Stop

Stop the METRICS_MONITOR service through ambari.

    module.exports = header: 'Ambari Metrics Monitor Stop', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Service Stop

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'METRICS_MONITOR'
        hostname: options.fqdn

