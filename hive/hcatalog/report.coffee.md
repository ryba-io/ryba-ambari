
# Hive HCatalog Info

Retrieve various info about the HCatalog Server and the Hive Server2.

    module.exports = header: 'Ambari Hive HCatalog Report', handler: ({options}) ->

## Wait

      @call 'ryba-ambari-takeover/hive/hcatalog/wait', once: true, options.wait

## Info FS Roots

List the current FS root locations for the Hive databases.

      @system.execute
        header: 'Info FS Roots'
        cmd: mkcmd.hdfs options.hdfs_krb5_user, "hive --service metatool -listFSRoot 2>/dev/nul"
      , (err, _, stdout) ->
        return if err
        count = 0
        for line in string.lines stdout
          continue unless /^hdfs:\/\//.test line
          @emit 'report', key: "FS Root #{++count}", value: line

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd
    string = require 'nikita/lib/misc/string'
