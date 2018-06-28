
# MapReduce JobHistoryServer Status

Check if the Job History Server is running. The process ID is located by default
inside "/var/run/hadoop-mapreduce/".

    module.exports = header: 'Mapreduce Ambari JHS Status', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'

## Service Status

      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HISTORYSERVER'
        hostname: options.fqdn
