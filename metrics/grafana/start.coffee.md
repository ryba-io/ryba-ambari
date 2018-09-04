
# Ambari Metrics Grafana Start

Start the METRICS_GRAFANA service through ambari.

    module.exports = header: 'Ambari Metrics Grafana Start', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Start

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_GRAFANA'
        hostname: options.fqdn

