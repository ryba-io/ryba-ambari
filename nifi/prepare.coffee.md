
# NiFi prepare

This module runs everything that Ambari does not do for us

    module.exports = header: 'NiFi Ambari Prepare', handler: (options) ->

## IPTables

      rules = [
        { chain: 'INPUT', jump: 'ACCEPT', dport: options.port, protocol: 'tcp', state: 'NEW', comment: "NiFi WebUI port" }
      ]
      @tools.iptables
        header: 'IPTables'
        rules: rules

## Layout

As of Ambari 2.6.2.2, Ambari automatically creates additionnal content repository dirs but does not do it with proveance repository

      @call header: 'Layout', ->
        for dir in options.additionnal_dirs
          @system.mkdir
            target: dir
            uid: options.user.name
            gid: options.group.name

## JRE Keystore

Gardian Sesame CACERT for Sesame auth in NiFi + Certs for Ranger policy refresh
            
      @call
        if: options.certs?
      , (_, cb) ->
        tmp_location = "/tmp/ryba_cacert_#{Date.now()}"
        @each options.certs, (opts, callback) ->
          {source, local, name} = opts.value
          @file.download
            header: 'download cacert'
            source: source
            target: "#{tmp_location}/cacert"
            local: true
          @java.keystore_add
            header: "add cacert to #{name}"
            keystore: options.truststore.target
            storepass: options.truststore.password
            caname: name
            cacert: "#{tmp_location}/cacert"
          @next callback
        @system.remove
          target: tmp_location
        @next cb