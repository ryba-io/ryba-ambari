
# NiFi Registry Configure

    module.exports = (service) ->
      options = service.options

## Iptables

      options.port ?= '61443'

## Dependencies

    {merge} = require 'nikita/lib/misc'

[nifi-properties]:https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#cluster-node-properties
