
# WebHCat

    module.exports =  header: 'WebHCat Install', handler: ({options}) ->

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

      # @system.execute
      #   header: 'Fix HDFS tmp'
      #   cmd: mkcmd.hdfs options.hdfs_krb5_user, """
      #   if hdfs dfs -test -d /tmp/hadoop-hcat; then exit 2; fi
      #   hdfs dfs -mkdir -p /tmp/hadoop-hcat
      #   hdfs dfs -chown HTTP:#{options.hadoop_group.name} /tmp/hadoop-hcat
      #   hdfs dfs -chmod -R 1777 /tmp/hadoop-hcat
      #   """
      #   code_skipped: 2



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
