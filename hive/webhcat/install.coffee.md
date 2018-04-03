
# WebHCat

    module.exports =  header: 'WebHCat Install', handler: (options) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register 'hdfs_upload', 'ryba/lib/hdfs_upload'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

## IPTables

| Service | Port  | Proto | Info                |
|---------|-------|-------|---------------------|
| webhcat | 50111 | http  | WebHCat HTTP server |

IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

      @tools.iptables
        header: 'IPTables'
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.webhcat_site['templeton.port'], protocol: 'tcp', state: 'NEW', comment: "WebHCat HTTP Server" }
        ]
        if: options.iptables.action is 'start'

## Startup

Install the "hadoop-yarn-resourcemanager" service, symlink the rc.d startup script
inside "/etc/init.d" and activate it on startup.

      # @call header: 'Service', ->
      #   @service 'hive-webhcat-server'
      #   @service 'pig'   # Upload .tar.gz
      #   @service 'sqoop' # Upload .tar.gz
      #   @hdp_select
      #     name: 'hive-webhcat'
      @system.tmpfs
        if_os: name: ['redhat','centos'], version: '7'
        mount: options.pid_dir
        uid: options.user.name
        gid: options.hadoop_group.name
        perm: '0750'

# ## Directories
# 
# Create file system directories for log and pid.
# 
#       @call header: 'Layout', ->
#         @system.mkdir
#           target: options.log_dir
#           uid: options.user.name
#           gid: options.hadoop_group.name
#           mode: 0o755
#         @system.mkdir
#           target: options.pid_dir
#           uid: options.user.name
#           gid: options.hadoop_group.name
#           mode: 0o755

# ## HDFS Tarballs
# 
# Upload the Pig, Hive and Sqoop tarballs inside the "/hdp/apps/$version"
# HDFS directory. Note, the parent directories are created by the
# "ryba-ambari-takeover/hadoop/hdfs_dn/layout" module.
# 
#       @call header: 'HDFS Tarballs', ->
#         @hdfs_upload (
#           for lib in ['pig', 'hive', 'sqoop']
#             source: "/usr/hdp/current/#{lib}-client/#{lib}.tar.gz"
#             target: "/hdp/apps/$version/#{lib}/#{lib}.tar.gz"
#             lock: "/tmp/ryba-#{lib}.lock"
#             krb5_user: options.hdfs_krb5_user
#         )

        # Avoid HTTP response
        # Permission denied: user=ryba, access=EXECUTE, inode=\"/tmp/hadoop-hcat\":HTTP:hadoop:drwxr-x---

      @system.execute
        header: 'Fix HDFS tmp'
        cmd: mkcmd.hdfs options.hdfs_krb5_user, """
        if hdfs dfs -test -d /tmp/hadoop-hcat; then exit 2; fi
        hdfs dfs -mkdir -p /tmp/hadoop-hcat
        hdfs dfs -chown HTTP:#{options.hadoop_group.name} /tmp/hadoop-hcat
        hdfs dfs -chmod -R 1777 /tmp/hadoop-hcat
        """
        code_skipped: 2

## SPNEGO

Copy the spnego keytab with restricitive permissions

      @system.copy
        header: 'SPNEGO'
        source: '/etc/security/keytabs/spnego.service.keytab'
        target: options.webhcat_site['templeton.kerberos.keytab']
        uid: options.user.name
        gid: options.hadoop_group.name
        mode: 0o0660


## Install Component

      @ambari.hosts.component_wait
        header: 'WEBHCAT_SERVER'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'WEBHCAT_SERVER'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'WEBHCAT_SERVER'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'WEBHCAT_SERVER'
        hostname: options.fqdn


## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'

## TODO: Check Hive

```
hdfs dfs -mkdir -p front1-webhcat/mytable
echo -e 'a,1\nb,2\nc,3' | hdfs dfs -put - front1-webhcat/mytable/data
hive
  create database testhcat location '/user/ryba/front1-webhcat';
  create table testhcat.mytable(col1 STRING, col2 INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
curl --negotiate -u : -d execute="use+testhcat;select+*+from+mytable;" -d statusdir="testhcat1" http://front1.hadoop:50111/templeton/v1/hive
hdfs dfs -cat testhcat1/stderr
hdfs dfs -cat testhcat1/stdout
```
