
# Hadoop HDFS JournalNode Start

Start the JournalNode service through ambari.

    module.exports = header: 'HDFS JN Ambari Start', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

Wait for the Kerberos server and Zookeeper server.

      @call 'masson/core/krb5_client/wait', options.wait_krb5_client
      @call 'ryba-ambari-takeover/zookeeper/server/wait', options.wait_zookeeper_server

## Service Start

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'JOURNALNODE'
        hostname: options.fqdn

