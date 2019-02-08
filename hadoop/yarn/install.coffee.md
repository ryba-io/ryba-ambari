
# Ambari Takeover

    module.exports = header: 'YARN Ambari Install', handler: ({options}) ->

## Dependencies

    {merge} = require 'nikita/lib/misc'
    fs = require 'fs'
    ssh2fs = require 'ssh2-fs'
    properties = require 'ryba/lib/properties'
