
# Ambari Logsearch Feeder Stop

Stop the METRICS_MONITOR service through ambari.

    module.exports = header: 'Ambari Infra Solr Stop', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Service Stop

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'INFRA_SOLR'
        hostname: options.fqdn

