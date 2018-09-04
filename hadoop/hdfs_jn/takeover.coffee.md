
# HAdoop HDFS JournalNode

Wait transactions to be synced

    module.exports = header: 'HDFS JN Ambari TakeOver', handler: ({options}) ->
    
      @call header: 'HDFS JN Wait Txns', handler: ->
        for host in options.hosts
          protocol = if options.hdfs_site['dfs.http.policy'] is 'HTTP_ONLY' then 'http' else 'https'
          address = options.hdfs_site["dfs.journalnode.#{protocol}-address"]
          [, port] = address.split ':'
          nameservice = "#{options.hdfs_site['dfs.nameservices']}"
          @wait.execute
            cmd: mkcmd.hdfs options.hdfs_krb5_user, "curl --negotiate -k -u : #{protocol}://#{host}:#{port}/jmx?qry=Hadoop:service=JournalNode,name=Journal-#{nameservice} | grep '\"CurrentLagTxns\" : 0'"

      @service.stop
        header: 'Stop'
        name: 'hadoop-hdfs-journalnode'
      @system.remove
        header: 'Remove systemd file'
        target: '/usr/lib/systemd/system/hadoop-hdfs-journalnode.service'
        code_skipped: 1
      @system.remove
        header: 'Remove initd file'
        target: '/etc/init.d/hadoop-hdfs-journalnode'
        code_skipped: 1
      @system.execute
        header: 'Daemon reload'
        cmd: 'systemctl daemon-reload;systemctl reset-failed'
        code_skipped: 1
        
## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'