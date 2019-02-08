
# Hadoop Core Configuration

    module.exports = (service) ->
      options = service.options
      options.fqdn ?= service.node.fqdn

## Identities

      options.hadoop_group = merge {}, service.deps.hdfs[0].options.hadoop_group, options.hadoop_group
      options.ssl_server = merge {}, service.deps.hdfs[0].options.ssl_server, options.ssl_server
      options.ssl_client = merge {}, service.deps.hdfs[0].options.ssl_client, options.ssl_client

## Validation

HDFS does not accept underscore "_" inside the hostname or it fails on startup
with the log message:

```
17/05/15 00:31:54 WARN hdfs.DFSUtil: Exception in creating socket address master_01.ambari.ryba:8020
java.lang.IllegalArgumentException: Does not contain a valid host:port authority: master_01.ambari.ryba:8020
```

      throw Error "Invalid Hostname: #{service.node.fqdn} should not contain \"_\"" if /_/.test service.node.fqdn

## Kerberos

      options.krb5 ?= {}
      options.krb5.realm ?= service.deps.krb5_client.options.etc_krb5_conf?.libdefaults?.default_realm
      # Admin Information
      options.krb5.admin ?= service.deps.krb5_client.options.admin[options.krb5.realm]
      # Spnego
      options.spnego ?= {}
      options.spnego.principal ?= "HTTP/#{service.node.fqdn}@#{options.krb5.realm}"
      options.spnego.keytab ?= '/etc/security/keytabs/spnego.service.keytab'

    path = require 'path'
    quote = require 'regexp-quote'
    {merge} = require 'nikita/lib/misc'
