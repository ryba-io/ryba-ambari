
# Hadoop ZKFC Status

Check if the ZKFC daemon is running. Gives the Ambari's REST API status.
Behind the scene Ambari's agent check the process ID which is located by default
inside "/var/run/hadoop/hdfs/hadoop-hdfs-zkfc.pid".

    module.exports = header: 'HDFS ZKFC Ambari Status', handler: ->

## Registry

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'

## Service Status

      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ZKFC'
        hostname: options.fqdn
