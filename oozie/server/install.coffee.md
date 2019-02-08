
# Oozie Server Install

Oozie source code and examples are located in "/usr/share/doc/oozie-$version".

The current version of Oozie doesnt supported automatic failover of the Yarn
Resource Manager. RM HA (High Availability) must be configure with manual
failover and Oozie must target the active node.

    module.exports = header: 'Oozie Server Install', handler: ({options}) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register 'hdfs_mkdir', 'ryba/lib/hdfs_mkdir'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

# ## IPTables

        @call if: options.db.engine is 'mysql', ->
          @service
            name: 'mysql'
          @service
            name: 'mysql-connector-java'

      # @call header: 'Layout Directories', ->
      #   @system.mkdir
      #     target: options.data_dir
      #     uid: options.user.name
      #     gid: options.hadoop_group.name
      #     mode: 0o0755
      #   @system.mkdir
      #     target: options.log_dir
      #     uid: options.user.name
      #     gid: options.hadoop_group.name
      #     mode: 0o0755
      #   @system.mkdir
      #     target: options.pid_dir
      #     uid: options.user.name
      #     gid: options.hadoop_group.name
      #     mode: 0o0755
      #   @system.mkdir
      #     target: options.tmp_dir
      #     uid: options.user.name
      #     gid: options.hadoop_group.name
      #     mode: 0o0755
      #   @system.mkdir
      #     target: "#{options.conf_dir}/action-conf"
      #     uid: options.user.name
      #     gid: options.hadoop_group.name
      #     mode: 0o0755
      #   # Set permission to action conf
      #   @system.execute
      #     cmd: """
      #     chown -R #{options.user.name}:#{options.hadoop_group.name} #{options.conf_dir}/action-conf
      #     """
      #     shy: true
        # Waiting for recursivity in @system.mkdir
        # @system.execute
        #   cmd: """
        #   chown -R #{options.user.name}:#{options.hadoop_group.name} /usr/lib/oozie
        #   chown -R #{options.user.name}:#{options.hadoop_group.name} #{options.data_dir}
        #   chown -R #{options.user.name}:#{options.hadoop_group.name} #{options.conf_dir} #/..
        #   chmod -R 755 #{options.conf_dir} #/..
        #   """

# ## Environment
# 
# Update the Oozie environment file "oozie-env.sh" located inside
# "/etc/oozie/conf".
# 
# Note, environment variables are grabed by oozie and translated into java
# properties inside "./bin/oozied.distro". Here's an extract:
# 
# 
# ```bash
# catalina_opts="-Doozie.home.dir=${OOZIE_HOME}";
# catalina_opts="${catalina_opts} -Doozie.config.dir=${OOZIE_CONFIG}";
# catalina_opts="${catalina_opts} -Doozie.log.dir=${OOZIE_LOG}";
# catalina_opts="${catalina_opts} -Doozie.data.dir=${OOZIE_DATA}";
# catalina_opts="${catalina_opts} -Doozie.instance.id=${OOZIE_INSTANCE_ID}"
# catalina_opts="${catalina_opts} -Doozie.config.file=${OOZIE_CONFIG_FILE}";
# catalina_opts="${catalina_opts} -Doozie.log4j.file=${OOZIE_LOG4J_FILE}";
# catalina_opts="${catalina_opts} -Doozie.log4j.reload=${OOZIE_LOG4J_RELOAD}";
# catalina_opts="${catalina_opts} -Doozie.http.hostname=${OOZIE_HTTP_HOSTNAME}";
# catalina_opts="${catalina_opts} -Doozie.admin.port=${OOZIE_ADMIN_PORT}";
# catalina_opts="${catalina_opts} -Doozie.http.port=${OOZIE_HTTP_PORT}";
# catalina_opts="${catalina_opts} -Doozie.https.port=${OOZIE_HTTPS_PORT}";
# catalina_opts="${catalina_opts} -Doozie.base.url=${OOZIE_BASE_URL}";
# catalina_opts="${catalina_opts} -Doozie.https.keystore.file=${OOZIE_HTTPS_KEYSTORE_FILE}";
# catalina_opts="${catalina_opts} -Doozie.https.keystore.pass=${OOZIE_HTTPS_KEYSTORE_PASS}";
# ```
# 
#       writes = [
#           match: /^export OOZIE_HTTPS_KEYSTORE_FILE=.*$/mg
#           replace: "export OOZIE_HTTPS_KEYSTORE_FILE=#{options.ssl.keystore.target}"
#           append: true
#         ,
#           match: /^export OOZIE_HTTPS_KEYSTORE_PASS=.*$/mg
#           replace: "export OOZIE_HTTPS_KEYSTORE_PASS=#{options.ssl.keystore.password}"
#           append: true
#         ,
#           match: /^export CATALINA_OPTS="${CATALINA_OPTS} -Djavax.net.ssl.trustStore=(.*)/m
#           replace: """
#           export CATALINA_OPTS="${CATALINA_OPTS} -Djavax.net.ssl.trustStore=#{options.ssl.truststore.target}"
#           """
#           append: true
#         ,
#           match: /^export CATALINA_OPTS="${CATALINA_OPTS} -Djavax.net.ssl.trustStorePassword=(.*)/m
#           replace: """
#           export CATALINA_OPTS="${CATALINA_OPTS} -Djavax.net.ssl.trustStorePassword=#{options.ssl.truststore.password}"
#           """
#           append: true
#         ]
      # @file.assert
      #   target: options.hadoop_lib_home
      #   filetype: 'directory'
      # @file.render
      #   header: 'Oozie Environment'
      #   target: "#{options.conf_dir}/oozie-env.sh"
      #   source: "#{__dirname}/../resources/oozie-env.sh.j2"
      #   local: true
      #   context: options
      #   write: writes
      #   uid: options.user.name
      #   gid: options.group.name
      #   mode: 0o0755
      #   backup: true

# # ExtJS
# 
# Install the ExtJS Javascript library as part of enabling the Oozie Web Console.
# 
#       @system.copy
#         header: 'ExtJS Library'
#         source: '/usr/share/HDP-oozie/ext-2.2.zip'
#         target: '/usr/hdp/current/oozie-server/libext/'

# # HBase credentials
# 
# Install the HBase Libs as part of enabling the Oozie Unified Credentials with HBase.
# 
#       @service
#         name: 'hbase'
#       @system.copy
#         header: 'HBase Libs'
#         source: '/usr/hdp/current/hbase-client/lib/hbase-common.jar'
#         target: '/usr/hdp/current/oozie-server/libserver/'

# LZO

Install the LZO compression library as part of enabling the Oozie Web Console.

      @call header: 'LZO', ->
        @call (_, callback) ->
          @service
            name: 'lzo-devel'
            relax: true
          , (err) ->
            @service.remove
              if: !!err
              name: 'lzo-devel'
            @next callback
        @call
          if: options.stack_version in ['2.5']
          header: "LZO #{options.stack_version}"
          handler: ->
            @service
              name: 'hadoop-lzo'
            @service
              name: 'hadoop-lzo-native'
            lzo_jar = null
            @system.execute
              cmd: 'ls /usr/hdp/current/share/lzo/*/lib/hadoop-lzo-*.jar'
            , (err, {status, stdout}) ->
              return if err
              lzo_jar = stdout.trim()
            @call ->
              @system.execute
                cmd: """
                # Remove any previously installed version
                rm /usr/hdp/current/oozie-server/libext/hadoop-lzo-*.jar
                # Copy lzo
                cp #{lzo_jar} /usr/hdp/current/oozie-server/libext/
                """
                unless_exists: "/usr/hdp/current/oozie-server/libext/#{path.basename lzo_jar}"
        @call
          if: options.stack_version in ['2.6']
          header: "LZO #{options.stack_version}"
          handler: ->
            @service
              name: 'hadooplzo'
            @service
              name: 'hadooplzo-native'
            lzo_jar = null
            @system.execute
              cmd: 'ls /usr/hdp/current/hadoop-client/lib/hadoop-lzo-*.jar | grep -m1 .jar'
            , (err, {status, stdout}) ->
              return if err
              lzo_jar = stdout.trim()
            @call ->
              @system.execute
                cmd: """
                # Remove any previously installed version
                rm /usr/hdp/current/oozie-server/libext/hadoop-lzo-*.jar
                # Copy lzo
                cp #{lzo_jar} /usr/hdp/current/oozie-server/libext/
                """
                unless_exists: "/usr/hdp/current/oozie-server/libext/#{path.basename lzo_jar}"

## MySQL Driver

Copy or symlink the MySQL JDBC driver JAR into the /var/lib/oozie/ directory following
(HDP documentation)[http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.2.0/HDP_Man_Install_v22/index.html#Item1.12.4.3]

      @system.link
        header: 'MySQL Driver'
        source: '/usr/share/java/mysql-connector-java.jar'
        target: '/usr/hdp/current/oozie-server/libext/mysql-connector-java.jar'

      # @call header: 'Configuration', ->
      #   @hconfigure
      #     target: "#{options.conf_dir}/oozie-site.xml"
      #     source: "#{__dirname}/../resources/oozie-site.xml"
      #     local: true
      #     properties: options.oozie_site
      #     uid: options.user.name
      #     gid: options.group.name
      #     mode: 0o0755
      #     merge: true
      #     backup: true
      #   @file
      #     target: "#{options.conf_dir}/oozie-default.xml"
      #     source: "#{__dirname}/../resources/oozie-default.xml"
      #     local: true
      #     backup: true
      #   @hconfigure
      #     target: "#{options.conf_dir}/hadoop-conf/core-site.xml"
      #     # local_default: true
      #     properties: options.hadoop_config
      #     uid: options.user.name
      #     gid: options.group.name
      #     mode: 0o0755
      #     backup: true


# ## Falcon Support
# 
#       @call
#         header: 'Falcon'
#         if: options.falcon?.enabled
#       , ->
#         @hconfigure
#           header: 'Hive Site'
#           if: options.falcon?.enabled
#           target: "#{options.conf_dir}/action-conf/hive.xml"
#           properties: 'hive.metastore.execute.setugi': 'true'
#           merge: true
#         @service
#           name: 'falcon'
#         @system.mkdir
#           target: '/tmp/falcon-oozie-jars'
#         # Note, the documentation mentions using "-d" option but it doesnt
#         # seem to work. Instead, we deploy the jar where "-d" default.
#         @system.execute
#           cmd: """
#           falconext=`ls /usr/hdp/current/falcon-client/oozie/ext/falcon-oozie-el-extension-*.jar`
#           if [ -f /usr/hdp/current/oozie-server/libext/`basename $falconext` ]; then exit 3; fi
#           rm -rf /tmp/falcon-oozie-jars/*
#           cp  /usr/hdp/current/falcon-client/oozie/ext/falcon-oozie-el-extension-*.jar \
#             /usr/hdp/current/oozie-server/libext
#           """
#           code_skipped: 3
# 
# ## HBase support
# 
#       @call
#         header: 'HBase'
#         if: options.hbase.enabled
#       , ->
#         @system.copy (
#           header: 'HBase Libs'
#           source: "/usr/hdp/current/hbase-client/lib/#{file}"
#           target: '/usr/hdp/current/oozie-server/libext/'
#         ) for file in [
#           'hbase-common.jar'
#           'hbase-client.jar'
#           'hbase-server.jar'
#           'hbase-protocol.jar'
#           'hbase-hadoop2-compat.jar'
#         ]

## Kerberos

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos'
        principal: options.oozie_site['oozie.service.HadoopAccessorService.kerberos.principal'].replace '_HOST', options.fqdn
        randkey: true
        keytab: options.oozie_site['oozie.service.HadoopAccessorService.keytab.file']
        uid: options.user.name
        gid: options.group.name


## SQL Database Creation

      @call header: 'SQL Database Creation', ->
        throw Error 'Database engine not supported' unless options.db.engine in ['mysql', 'mariadb', 'postgresql']
        escape = (text) -> text.replace(/[\\"]/g, "\\$&")
        version_local = db.cmd(options.db, "select data from OOZIE_SYS where name='oozie.version'") + "| tail -1"
        version_remote = "ls /usr/hdp/current/oozie-server/lib/oozie-client-*.jar | sed 's/.*client\\-\\(.*\\).jar/\\1/'"
        @db.user options.db, database: null,
          header: 'User'
        @db.database options.db,
          header: 'Database'
          user: options.db.username
        @db.schema options.db,
          header: 'Schema'
          if: options.db.engine is 'postgresql'
          schema: options.db.schema or options.db.database
          database: options.db.database
          owner: options.db.username
          
## SPNEGO

Ensure we have read access to the spnego keytab soring the server HTTP
principal.

      @system.execute
        header: 'SPNEGO'
        cmd: "su -l #{options.user.name} -c 'test -r /etc/security/keytabs/spnego.service.keytab'"

# User limits

      @system.limits
        header: 'Ulimit'
        user: options.user.name
      , options.user.limits

# Install Component

      @ambari.hosts.component_wait
        header: 'OOZIE_SERVER WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'OOZIE_SERVER'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'OOZIE_SERVER INSTALL'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'OOZIE_SERVER'
        hostname: options.fqdn
      # 
      # @call ->
      #   @system.execute
      #      cmd: "su -l #{options.user.name} -c '/usr/hdp/current/oozie-server/bin/ooziedb.sh create -sqlfile /tmp/oozie.sql -run Validate DB Connection'"
      #      unless_exec: db.cmd options.db, "select data from OOZIE_SYS where name='oozie.version'"
      #   @system.execute
      #      cmd: "su -l #{options.user.name} -c '/usr/hdp/current/oozie-server/bin/ooziedb.sh upgrade -run'"
      #      unless_exec: "[[ `#{version_local}` == `#{version_remote}` ]]"
      # 
      # @call header: 'War', ->
      #   @system.execute
      #     header: 'Stop before WAR'
      #     cmd: """
      #     if [ ! -f #{options.pid_dir}/oozie.pid ]; then exit 3; fi
      #     if ! kill -0 >/dev/null 2>&1 `cat #{options.pid_dir}/oozie.pid`; then exit 3; fi
      #     su -l #{options.user.name} -c "/usr/hdp/current/oozie-server/bin/oozied.sh stop 20 -force"
      #     rm -rf cat #{options.pid_dir}/oozie.pid
      #     """
      #     code_skipped: 3
      #   # The script `ooziedb.sh` must be done as the oozie Unix user, otherwise
      #   # Oozie may fail to start or work properly because of incorrect file permissions.
      #   # There is already a "oozie.war" file inside /var/lib/oozie/oozie-server/webapps/.
      #   # The "prepare-war" command generate the file "/var/lib/oozie/oozie-server/webapps/oozie.war".
      #   # The directory being served by the web server is "prepare-war".
      #   # See note 20 lines above about "-d" option
      #   # falcon_opts = if falcon_ctxs.length then " –d /tmp/falcon-oozie-jars" else ''
      #   secure_opt = if options.ssl.enabled then '-secure' else ''
      #   falcon_opts = ''
      #   @system.execute
      #     header: 'Prepare WAR'
      #     cmd: """
      #     chown #{options.user.name} /usr/hdp/current/oozie-server/oozie-server/conf/server.xml
      #     su -l #{options.user.name} -c 'cd /usr/hdp/current/oozie-server; ./bin/oozie-setup.sh prepare-war #{secure_opt} #{falcon_opts}'
      #     """
      #     code_skipped: 255 # Oozie already started, war is expected to be installed

## Dependencies

    url = require 'url'
    path = require 'path'
    mkcmd = require 'ryba/lib/mkcmd'
    db = require 'nikita/lib/misc/db'
    quote = require 'regexp-quote'
