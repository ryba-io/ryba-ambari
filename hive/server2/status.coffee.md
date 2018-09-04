
# Hive Server2 Status

Check if the Hive Server2 is running. The process ID is located by default
inside "/var/run/hive/hive-server2.pid".

Exit code is "3" if server not runnig or "1" if server not running but pid file
still exists.

    module.exports = header: 'Ambari Hive Server2 Status', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'


## Curl

      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE_SERVER'
        hostname: options.fqdn
