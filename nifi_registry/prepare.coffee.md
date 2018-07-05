
# NiFi Registry prepare

This module runs everything that Ambari does not do for us

    module.exports = header: 'NiFi Registry Ambari Prepare', handler: (options) ->

## IPTables

      rules = [
        { chain: 'INPUT', jump: 'ACCEPT', dport: options.port, protocol: 'tcp', state: 'NEW', comment: "NiFi Registry WebUI port" }
      ]
      @tools.iptables
        header: 'IPTables'
        rules: rules