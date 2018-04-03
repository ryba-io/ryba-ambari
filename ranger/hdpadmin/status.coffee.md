
# Ranger Admin Status

Check if Ranger Admin is started

    module.exports = header: 'Ambari Ranger Admin Status', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'

## Service Status

      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'RANGER_ADMIN'
        hostname: options.fqdn
