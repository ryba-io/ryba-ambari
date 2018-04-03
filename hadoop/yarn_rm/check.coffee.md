
# Hadoop Yarn ResourceManager Check

Check the health of the ResourceManager(s).

    module.exports = header: 'YARN RM Ambari Check', handler: (options) ->

## Wait

Wait for the ResourceManager.

      @call once: true, 'ryba-ambari-takeover/hadoop/yarn_rm/wait', options.wait

## Check Health

Connect to the provided ResourceManager to check its health. This command
`yarn rmadmin -checkHealth {serviceId}` return 0 if the ResourceManager is
healthy, non-zero otherwise. This check only apply to High Availability
mode.

      @system.execute
        header: 'HA Health'
        if: options.yarn_site['yarn.resourcemanager.ha.enabled'] is 'true'
        cmd: """
            kinit -kt #{options.yarn_site['yarn.resourcemanager.keytab']} -p #{options.yarn_site['yarn.resourcemanager.principal'].replace '_HOST', options.fqdn}
            yarn --config #{options.hadoop_conf_dir} rmadmin -checkHealth #{options.hostname}
          """
        # cmd: mkcmd.hdfs options.hdfs_krb5_user, "yarn --config #{options.hadoop_conf_dir} rmadmin -checkHealth #{options.hostname}"
        retry: 3
        wait: 5000

# Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
