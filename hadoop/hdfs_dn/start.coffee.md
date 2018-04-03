
# Hadoop HDFS Datanode Start

Start the JournalNode service through ambari.

    module.exports = header: 'HDFS DN Ambari Start', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

Wait for the Kerberos server and Zookeeper server.

      @call once: true, 'masson/core/krb5_client/wait', options.wait_krb5_client

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'DATANODE'
        hostname: options.fqdn

