
# HAdoop HDFS JournalNode

Wait transactions to be synced

    module.exports = header: 'HDFS NN Ambari TakeOver', handler: ({options}) ->
      @service.stop
        header: 'Stop'
        name: 'hadoop-hdfs-namenode'
      @system.remove
        if_os: name: ['redhat','centos'], version: '7'
        header: 'Remove systemd file'
        target: '/usr/lib/systemd/system/hadoop-hdfs-namenode.service'
        code_skipped: 1
      @system.remove
        if_os: name: ['redhat','centos'], version: '6'
        header: 'Remove initd file'
        target: '/etc/init.d/hadoop-hdfs-namenode'
        code_skipped: 1
      @service.stop
        header: 'Stop'
        name: 'hadoop-hdfs-zkfc'
      # @system.remove
      #   if_os: name: ['redhat','centos'], version: '7'
      #   header: 'Remove systemd file'
      #   target: '/usr/lib/systemd/system/hadoop-hdfs-zkfc.service'
      #   code_skipped: 1
      # @system.remove
      #   if_os: name: ['redhat','centos'], version: '6'
      #   header: 'Remove initd file'
      #   target: '/etc/init.d/hadoop-hdfs-zkfc'
      #   code_skipped: 1
      # @system.execute
      #   if_os: name: ['redhat','centos'], version: '7'
      #   header: 'Daemon reload'
      #   cmd: 'systemctl daemon-reload;systemctl reset-failed'
      #   code_skipped: 1
