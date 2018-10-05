
# Hadoop HDFS NameNode Status

Check if the HDFS NameNode server is running. The process ID is located by default
inside "/var/run/hadoop-hdfs/hdfs/hadoop-hdfs-namenode.pid".

    module.exports = header: 'HDFS NN Ambari Status', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'

## Service Status

      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'NAMENODE'
        hostname: options.fqdn
