
# Graceful Stop For HBase regionserver

    module.exports = header: 'Graceful Stop HBase Regionserver', handler: ({options}) ->

## Steps

      @system.execute
        cmd: mkcmd.hbase options.admin, """
        /usr/hdp/current/hbase-regionserver/bin/graceful_stop.sh --config /etc/hbase-regionserver/conf --maxthreads 32 #{options.fqdn}
        """
      @service.stop
        name: 'hbase-regionserver'
      @system.execute
        cmd: mkcmd.hbase options.admin, """
        echo 'balance_switch true; balancer' | hbase --config /etc/hbase-regionserver/conf/ shell
        """
      @system.execute
        cmd: 'rm -f /etc/init.d/hbase-regionserver'
        code_skipped: 1
      @system.execute
        cmd: 'rm -f /usr/lib/systemd/system/hbase-regionserver.service'
        code_skipped: 1
      @system.execute
        header: 'Daemon reload'
        cmd: 'systemctl daemon-reload;systemctl reset-failed'
        code_skipped: 1

      
    mkcmd = require 'ryba/lib/mkcmd'
