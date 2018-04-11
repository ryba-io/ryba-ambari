
# Ambari Logsearch Feeder

Start the Metrics  Monitor service through ambari.

    module.exports = header: 'Ambari Logsearch Feeder Start', handler: (options) ->
      
## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

      @call 'ryba-ambari-takeover/logsearch/server/wait', options.wait_logsearch_server

## Start

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'LOGSEARCH_LOGFEEDER'
        hostname: options.fqdn

