
# Hadoop ZKFC Start

Start the NameNode service as well as its ZKFC daemon.

In HA mode, to ensure that the leadership is assigned to the desired active
NameNode, the ZKFC daemons on the standy NameNodes wait for the one on the
active NameNode to start first.

    module.exports = header: 'HDFS ZKFC Ambari Start', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'


      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ZKFC'
        hostname: options.fqdn

## Wait ZKFC

      @connection.wait options.wait
        
## Wait Failover

Ensure a given NameNode is always active and force the failover otherwise.

In order to work properly, the ZKFC daemon must be running and the command must
be executed on the same server as ZKFC.

      # Note, probably we shall wait for the other NameNode to be started and running
      # before attempting to activate it.
      @system.execute
        header: 'Failover'
        cmd: mkcmd.hdfs options.hdfs_krb5_user, """
        if hdfs --config #{options.nn_conf_dir} haadmin -getServiceState #{options.active_shortname} | grep standby;
        then hdfs --config #{options.nn_conf_dir} haadmin -ns #{options.dfs_nameservices} -failover #{options.standby_shortname} #{options.active_shortname};
        else exit 2; fi
        """
        code_skipped: 2

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
