
# Apache Atlas Install

    module.exports = header: 'Ambari Atlas Install', handler: ({options}) ->
      protocol = if options.application.properties['atlas.enableTLS'] is 'true' then 'https' else 'http'
      credential_file = options.application.properties['cert.stores.credential.provider.path'].split('jceks://file')[1]
      credential_name = path.basename credential_file
      credential_dir = path.dirname credential_file

## Registry

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['file', 'jaas'], 'ryba/lib/file_jaas'
      @registry.register 'ranger_service', 'ryba/ranger/actions/ranger_service'
      @registry.register 'ranger_policy', 'ryba/ranger/actions/ranger_policy'
      @registry.register 'ranger_service_wait', 'ryba/ranger/actions/ranger_service_wait'
      @registry.register 'ranger_user', 'ryba/ranger/actions/ranger_user'
      console.log options.application.properties['cert.stores.credential.provider.path']

## Wait

      # @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      # @call 'ryba/zookeeper/server/wait', once: true, options.wait_zookeeper_server
      # @call 'ryba/hbase/master/wait', once: true, options.wait_hbase
      # @call 'ryba/kafka/broker/wait', once: true, options.wait_kafka
      # @call 'ryba/ranger/admin/wait', once: true, options.wait_ranger

## Identities

      @system.group header: 'Group',  options.group
      @system.user header: 'User', options.user

## IPTables
IPTables rules are only inserted if the parameter "iptables.action" is set to
"start" (default value).

| Service       | Port   | Proto        | Parameter |
|---------------|--------|--------------|-----------|
| Atlas Server  | 21000  | http         | port      |
| Atlas Server  | 21443  | https        | port      |


      @tools.iptables
        if: options.iptables
        rules: [
          { chain: 'INPUT', jump: 'ACCEPT', dport: options.application.properties["atlas.server.#{protocol}.port"], protocol: 'tcp', state: 'NEW', comment: "Atlas Server #{protocol}" }
        ]

## Package & Repository

Install Atlas packages

      @service
        header: 'Atlas Package'
        name: 'atlas-metadata'
      @hdp_select
        name: 'atlas-server'
      @hdp_select
        name: 'atlas-client'

## SSL 

Import certificates, private and public keys of the host.

      @java.keystore_add
        keystore: options.application.properties['keystore.file']
        storepass: options.ssl.keystore.password
        key: options.ssl.key.source
        cert: options.ssl.cert.source
        keypass: options.ssl.keystore.keypass
        name: options.ssl.key.name
        local: options.ssl.cert.local
        uid: options.user.name
        gid: options.group.name
        mode: 0o0640
      @java.keystore_add
        keystore: options.application.properties['keystore.file']
        storepass: options.ssl.keystore.password
        caname: "hadoop_root_ca"
        cacert: options.ssl.cacert.source
        local: options.ssl.cacert.local
      @java.keystore_add
        keystore: options.application.properties['truststore.file']
        storepass: options.ssl.truststore.password
        caname: "hadoop_root_ca"
        cacert: options.ssl.cacert.source
        local: options.ssl.cacert.local
        uid: options.user.name
        gid: options.group.name
        mode: 0o0644
      @call
        # if: -> @status(-3) or @status(-2)
        header: 'Generate Credentials SSL provider file'
      , (_, callback) ->
        ssh = @ssh options.ssh
        ssh.shell (err, stream) =>
          stream.write 'if /usr/hdp/current/atlas-client/bin/cputil.py ;then exit 0; else exit 1;fi\n'
          data = ''
          error = exit = null
          stream.on 'data', (data, extended) =>
            data = data.toString()
            switch
              when /Please enter the full path to the credential provider:/.test data
                options.log "prompt: #{data}"
                options.log "writing: #{options.application.properties['cert.stores.credential.provider.path']}\n"
                stream.write "#{options.application.properties['cert.stores.credential.provider.path']}\n"
                data = ''
              when /Please enter the password value for keystore.password:/.test data
                options.log "prompt: #{data}"
                options.log "write: #{options.ssl.keystore.password}"
                stream.write "#{options.ssl.keystore.password}\n"
                data = ''
              when /Please enter the password value for keystore.password again:/.test data
                options.log "prompt: #{data}"
                options.log "write: #{options.ssl.keystore.password}"
                stream.write "#{options.ssl.keystore.password}\n"
                data = ''
              when /Please enter the password value for truststore.password:/.test data
                options.log "prompt: #{data}"
                options.log "write: #{options.ssl.truststore.password}"
                stream.write "#{options.ssl.truststore.password}\n"
                data = ''
              when /Please enter the password value for truststore.password again:/.test data
                options.log "prompt: #{data}"
                options.log "write: #{options.ssl.truststore.password}"
                stream.write "#{options.ssl.truststore.password}\n"
                data = ''
              when /Please enter the password value for password:/.test data
                options.log "prompt: #{data}"
                options.log "write: #{options.ssl.keystore.keypass}"
                stream.write "#{options.ssl.keystore.keypass}\n"
                data = ''
              when /Please enter the password value for password again:/.test data
                options.log "prompt: #{data}"
                options.log "write: #{options.ssl.keystore.keypass}"
                stream.write "#{options.ssl.keystore.keypass}\n"
                data = ''
              when /Entry for keystore.password already exists/.test data
                stream.write "y\n"
                data = ''
              when /Entry for truststore.password already exists/.test data
                stream.write "y\n"
                data = ''
              when /Entry for password already exists/.test data
                stream.write "y\n"
                data = ''
              when /Exception in thread.*/.test data
                error = new Error data
                stream.end 'exit\n' unless exit
                exit = true
          stream.on 'exit', =>
            return callback error if error
            callback null, true
      @system.chown
        header: 'Ownership credential'
        target: "#{credential_dir}/#{credential_name}"
        uid: options.user.name
        gid: options.group.name
        mode: 0o770
      @system.chown
        header: 'Ownership crc'
        target: "#{credential_dir}/.#{credential_name}.crc"
        uid: options.user.name
        gid: options.group.name
        mode: 0o770

## Kerberos

Add The Kerberos Principal for atlas service and setup a JAAS configuration file
for atlas to able to open client connection to solr for its indexing backend.

      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos Atlas Service'
        randkey: true
        principal: options.application.properties['atlas.authentication.principal'].replace '_HOST', options.fqdn
        keytab: options.application.properties['atlas.authentication.keytab']
        uid: options.user.name
        gid: options.group.name
        mode: 0o660
      # @krb5.addprinc options.krb5.admin,
      #   header: 'Kerberos Atlas Service'
      #   principal: options.application.properties['atlas.http.authentication.kerberos.principal'].replace '_HOST', options.fqdn
      #   randkey: true
      #   keytab: options.application.properties['atlas.http.authentication.kerberos.keytab']
      #   uid: 'root'
      #   gid: options.hadoop_group.name
      #   mode: 0o660
      #   unless: -> @status -1
#       @file.jaas
#         if: options.atlas_opts['java.security.auth.login.config']?
#         header: 'Atlas Service JAAS'
#         target: options.atlas_opts['java.security.auth.login.config']
#         mode: 0o750
#         uid: options.user.name
#         gid: options.group.name
#         content:
#           KafkaClient:
#             principal: options.application.properties['atlas.authentication.principal']
#             keyTab: options.application.properties['atlas.authentication.keytab']
#             useKeyTab: true
#             storeKey: true
#             serviceName: 'kafka'
#             useTicketCache: true
#           Client:
#             useKeyTab: true
#             storeKey: true
#             useTicketCache: false
#             doNotPrompt: false
#             keyTab: options.application.properties['atlas.authentication.keytab']
#             principal: options.application.properties['atlas.authentication.principal'].replace '_HOST', options.fqdn
      @krb5.addprinc options.krb5.admin,
        header: 'Kerberos Atlas Service Admin Users'
        principal: options.admin_principal
        randkey: true
        password: options.admin_password

## Deploy Atlas War

Need to copy the atlas war file if `env['ATLAS_EXPANDED_WEBAPP_DIR']` is
set to other than the default

## Setup Credentials File

Convert the user_creds object into a file of credentials. See [how to generate][atlas-credential-file] atlas
credential based on file.

```cson
  user_creds
    'toto':
      name: 'toto'
      password: 'toto123'
      group: 'user'
    'juju':
      name: 'julie'
      password: 'juju123'
      group: 'user'
```

      @call
        if: options.application.properties['atlas.authentication.method.file'] is 'true'
        header: 'Render Credentials file'
      , ->
        old_lines = []
        new_lines = []
        content = ''
        @call header: 'Read Current Credential', (_, callback )  ->
          ssh = @ssh options.ssh
          fs.readFile ssh, options.application.properties['atlas.authentication.method.file.filename'], 'utf8', (err, content) ->
            return callback null, true if err and err.code is 'ENOENT'
            return callback err if err
            old_lines = string.lines content
            return if old_lines.length > 0 then callback null, true else callback null, false
        @call
          header: 'Merge user credentials'
          if: -> @status -1
        , ->
          for line in old_lines
            name = line.split(':')[0]
            new_lines.push unless name in Object.keys(options.user_creds)#keep track of old user if not present in current config
        @call header: 'Generate credential file', ->
          @each options.user_creds, ({options}, callback) ->
            name = options.key
            user = options.value
            line = "#{user.name}=#{user.group}"
            @system.execute
              header: 'Generate new credential'
              cmd: "echo -n '#{user.password}' | sha256sum"
            ,(err, {status, stdout}) ->
              throw err if err
              [match] = /[a-zA-Z0-9]*/.exec stdout.trim()
              new_lines.push "#{line}::#{match}"
            @next callback
          @call ->
            @file
              content: new_lines.join "/n"
              target: options.application.properties['atlas.authentication.method.file.filename']
              mode: 0o740
              eof: true
              backup: true
              uid: options.user.name
              gid: options.user.name

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
    string = require 'nikita/lib/misc/string'
    path = require 'path'
    fs = require 'ssh2-fs'
    {merge} = require 'nikita/lib/misc'

[atlas-credential-file]:(https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.0/bk_data-governance/content/ch_hdp_data_governance_install_atlas_ambari.html)
[solr-rest-api-roles]:(https://lucene.apache.org/solr/guide/6_6/rule-based-authorization-plugin.html)
