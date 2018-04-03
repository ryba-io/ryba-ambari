

# Capacity Scheduler

The [CapacityScheduler][capacity], a pluggable scheduler for Hadoop which allows for
multiple-tenants to securely share a large cluster such that their applications
are allocated resources in a timely manner under constraints of allocated
capacities

Note about the property "yarn.scheduler.capacity.resource-calculator": The
default i.e. "org.apache.hadoop.yarn.util.resource.DefaultResourseCalculator"
only uses Memory while DominantResourceCalculator uses Dominant-resource to
compare multi-dimensional resources such as Memory, CPU etc. A Java
ResourceCalculator class name is expected.

    module.exports = header: 'YARN RM Ambari Sheduler', handler: (options) ->

## Reload

      @system.execute
        header: 'Reload'
        if: -> @status -1
        cmd: mkcmd.hdfs options.hdfs_krb5_user, "service hadoop-yarn-resourcemanager status && yarn --config #{options.conf_dir} rmadmin -refreshQueues || exit 0"


## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
