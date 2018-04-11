
# MapReduce Install

    module.exports = header: 'MapReduce Client Install', handler: (options) ->

## Register

      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register 'hdfs_upload', 'ryba/lib/hdfs_upload'

## IPTables

| Service    | Port        | Proto | Parameter                                   |
|------------|-------------|-------|---------------------------------------------|
| mapreduce  | 59100-59200 | http  | yarn.app.mapreduce.am.job.client.port-range |


IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      jobclient = options.mapred_site['yarn.app.mapreduce.am.job.client.port-range']
      jobclient = jobclient.replace '-', ':'
      @tools.iptables
        header: 'IPTables'
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: jobclient, protocol: 'tcp', state: 'NEW', comment: "MapRed Client Range" }
        ]

## Identities

      @system.group header: 'Group', options.hadoop_group
      @system.user header: 'User', options.user

## Service

      @call header: 'Packages', ->
        @service
          name: 'hadoop-mapreduce'
        @hdp_select
          name: 'hadoop-client'

## HDFS Tarballs

Upload the MapReduce tarball inside the "/hdp/apps/$version/mapreduce"
HDFS directory. Note, the parent directories are created by the
"ryba-ambari-takeover/hadoop/hdfs_dn/layout" module.

      @hdfs_upload
        header: 'HDFS Tarballs'
        wait: 60*1000
        source: '/usr/hdp/current/hadoop-client/mapreduce.tar.gz'
        target: '/hdp/apps/$version/mapreduce/mapreduce.tar.gz'
        id: options.hostname
        lock: '/tmp/ryba-mapreduce.lock'
        krb5_user: options.hdfs_krb5_user

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

### MAPREDUCE2_CLIENT component wait
Wait for the MAPREDUCE2_CLIENT component to be declared on the host

      @ambari.hosts.component_wait
        header: 'MAPREDUCE2_CLIENT WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'MAPREDUCE2_CLIENT'
        hostname: options.fqdn

### MAPREDUCE2_CLIENT component install
Put the MAPREDUCE2_CLIENT component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        if: options.takeover
        header: 'MAPREDUCE2_CLIENT set installed'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'MAPREDUCE2_CLIENT'
        hostname: options.fqdn

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
