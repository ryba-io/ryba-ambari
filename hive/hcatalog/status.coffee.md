
# Hive HCatalog Status

Check if the HCatalog is running. The process ID is located by default
inside "/var/run/hive-hcatalog/hive-hcatalog.pid".

    module.exports = header: 'Ambari Hive HCatalog Status', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'

## Ambari Agent's status

      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE_METASTORE'
        hostname: options.fqdn