
# Hadoop ZKFC Install

    module.exports = header: 'HDFS ZKFC Ambari Install', handler: ({options}) ->

## Register

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['file', 'jaas'], 'ryba/lib/file_jaas'
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## ZK Auth and ACL

Secure the Zookeeper connection with JAAS. In a Kerberos cluster, the SASL
provider is configured with the NameNode principal. The digest provider may also
be configured if the property "ryba.zkfc.digest.password" is set.

The permissions for each provider is "cdrwa", for example:

```
sasl:nn:cdrwa
digest:hdfs-zkfcs:KX44kC/I5PA29+qXVfm4lWRm15c=:cdrwa
```

Note, we didnt test a scenario where the cluster is not secured and the digest
isn't set. Probably the default acl "world:anyone:cdrwa" is used.

http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html#Securing_access_to_ZooKeeper

If you need to change the acl manually inside zookeeper, you can use this
command as an example:

```
setAcl /hadoop-ha sasl:zkfc:cdrwa,sasl:nn:cdrwa,digest:zkfc:ePBwNWc34ehcTu1FTNI7KankRXQ=:cdrwa
```

      @call header: 'ZK Auth and ACL', ->
        acls = []
        # acls.push 'world:anyone:r'
        jaas_user = /^(.*?)[@\/]/.exec(options.principal)?[1]
        acls.push "sasl:#{jaas_user}:cdrwa" if options.core_site['hadoop.security.authentication'] is 'kerberos'
        @file
          target: "/etc/security/zookeeper/zk-auth.txt"
          content: if options.digest.password then "digest:#{options.digest.name}:#{options.digest.password}" else ""
          uid: options.user.name
          gid: options.group.name
          mode: 0o0700
        @system.execute
          cmd: """
          export ZK_HOME=/usr/hdp/current/zookeeper-client/
          java -cp $ZK_HOME/lib/*:$ZK_HOME/zookeeper.jar org.apache.zookeeper.server.auth.DigestAuthenticationProvider #{options.digest.name}:#{options.digest.password}
          """
          shy: true
          if: !!options.digest.password
        , (err, {status, stdout}) ->
          throw err if err
          return unless stdout
          digest = match[1] if match = /\->(.*)/.exec(stdout)
          throw Error "Failed to get digest" unless digest
          acls.push "digest:#{digest}:cdrwa"
        @call ->
          @file
            target: "/etc/security/zookeeper/zk-acl.txt"
            content: acls.join ','
            uid: options.user.name
            gid: options.group.name
            mode: 0o0600

# ## SSH Fencing
#
# Implement the SSH fencing strategy on each NameNode. To achieve this, the
# "hdfs-site.xml" file is updated with the "dfs.ha.fencing.methods" and
# "dfs.ha.fencing.ssh.private-key-files" properties.
#
# For SSH fencing to work, the HDFS user must be able to log for each NameNode
# into any other NameNode. Thus, the public and private SSH keys of the
# HDFS user are deployed inside his "~/.ssh" folder and the
# "~/.ssh/authorized_keys" file is updated accordingly.
#
# We also make sure SSH access is not blocked by a rule defined
# inside "/etc/security/access.conf". A specific rule for the HDFS user is
# inserted if ALL users or the HDFS user access is denied.
#
#       @call
#         header: 'SSH Fencing'
#       , ->
#         @system.mkdir
#           target: "#{options.user.home}/.ssh"
#           uid: options.user.name
#           gid: options.hadoop_group.name
#           mode: 0o700
#         @file.download
#           source: "#{options.ssh_fencing.private_key}"
#           target: "#{options.user.home}/.ssh/id_rsa"
#           uid: options.user.name
#           gid: options.hadoop_group.name
#           mode: 0o600
#         @file.download
#           source: "#{options.ssh_fencing.public_key}"
#           target: "#{options.user.home}/.ssh/id_rsa.pub"
#           uid: options.user.name
#           gid: options.hadoop_group.name
#           mode: 0o644
#         @call (_, callback) ->
#           fs.readFile null, "#{options.ssh_fencing.public_key}", (err, content) =>
#             return callback err if err
#             @file
#               target: "#{options.user.home}/.ssh/authorized_keys"
#               content: content
#               append: true
#               uid: options.user.name
#               gid: options.hadoop_group.name
#               mode: 0o600
#             , (err, written) =>
#               return callback err if err
#               ssh = @ssh options.ssh
#               fs.readFile ssh, '/etc/security/access.conf', 'utf8', (err, source) =>
#                 return callback err if err
#                 content = []
#                 # exclude = ///^\-\s?:\s?(ALL|#{options.user.name})\s?:\s?(.*?)\s*?(#.*)?$///
#                 # include = ///^\+\s?:\s?(#{options.user.name})\s?:\s?(.*?)\s*?(#.*)?$///
#                 exclude = /^\-\s?:\s?(ALL|#{options.user.name})\s?:\s?(.*?)\s*?(#.*)?$/
#                 include = /^\+\s?:\s?(#{options.user.name})\s?:\s?(.*?)\s*?(#.*)?$/
#                 included = false
#                 for line, i in source = source.split /\r\n|[\n\r\u0085\u2028\u2029]/g
#                   if match = include.exec line
#                     included = true # we shall also check if the ip/fqdn match in origin
#                   if not included and match = exclude.exec line
#                     content.push "+ : #{options.user.name} : #{options.nn_hosts}"
#                   content.push line
#                 return callback null, false if content.length is source.length
#                 @file
#                   target: '/etc/security/access.conf'
#                   content: content.join '\n'
#                 .next callback

# ## HA Auto Failover
# 
# The action start by enabling automatic failover in "hdfs-site.xml" and configuring HA zookeeper quorum in
# "core-site.xml". The impacted properties are "dfs.ha.automatic-failover.enabled" and
# "ha.zookeeper.quorum". Then, we wait for all ZooKeeper to be started. Note, this is a requirement.
# 
# If this is an active NameNode, we format ZooKeeper and start the ZKFC daemon. If this is a standby
# NameNode, we wait for the active NameNode to take leadership and start the ZKFC daemon.
# 
#       # @call 'ryba-ambari-takeover/zookeeper/server/wait', once: true, options.wait_zookeeper_server
# 
#       @system.execute
#         header: 'Format ZK'
#         if: [
#           -> options.active_nn_host is options.fqdn
#           -> options.automatic_failover
#         ]
#         cmd: "yes n | hdfs --config #{options.conf_dir} zkfc -formatZK"
#         code_skipped: 2


## Dependencies

    fs = require 'ssh2-fs'
    mkcmd = require 'ryba/lib/mkcmd'
    {merge} = require 'nikita/lib/misc'
