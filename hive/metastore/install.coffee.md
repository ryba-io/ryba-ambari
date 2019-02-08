
# Hive Metastore Install

    module.exports =  header: 'Ambari Hive Metastore Install', handler: ({options}) ->

## Dependencies

    db = require 'nikita/lib/misc/db'
    {merge} = require 'nikita/lib/misc'
