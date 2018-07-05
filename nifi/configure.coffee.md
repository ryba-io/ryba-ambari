
# NiFi Configure

    module.exports = (service) ->
      options = service.options
      
## Identities

      # Group
      options.group = name: options.group if typeof options.group is 'string'
      options.group ?= {}
      options.group.name ?= 'nifi'
      options.group.system ?= true
      # User
      options.user = name: options.user if typeof options.user is 'string'
      options.user ?= {}
      options.user.name ?= 'nifi'
      options.user.gid = options.group.name
      options.user.system ?= true
      options.user.comment ?= 'NiFi User'
      options.user.home ?= '/var/lib/nifi'
      options.user.limits ?= {}
      options.user.limits.nofile ?= 64000
      options.user.limits.nproc ?= 10000

## Additional Dirs

      additionnal_dirs = []

## Iptables

      options.port ?= '9760'

## Keystore

      options.certs ?= {}
      options.truststore ?= {}
      options.truststore.target ?= '/usr/java/latest/jre/lib/security/cacerts'
      options.truststore.password ?= 'changeit'

## Dependencies

    {merge} = require 'nikita/lib/misc'

[nifi-properties]:https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#cluster-node-properties
