
# MapReduce Install

    module.exports = header: 'MapReduce Client Install', handler: ({options}) ->

## Register

      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register 'hdfs_upload', 'ryba/lib/hdfs_upload'


## Ulimit

Increase ulimit for the MapReduce user. The HDP package create the following
files:

```bash
cat /etc/security/limits.d/mapred.conf
mapred    - nofile 32768
mapred    - nproc  65536
```

Note, a user must re-login for those changes to be taken into account. See
the "ryba-ambari-takeover/hadoop/hdfs" module for additional information.

      @system.limits
        header: 'Ulimit'
        user: options.user.name
      , options.user.limits


## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
