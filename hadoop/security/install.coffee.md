
# Hadoop Core Install

    module.exports = header: 'Hadoop Security Install', handler: ({options}) ->

## Keytab Directory

      @system.mkdir
        header: 'Keytabs'
        target: '/etc/security/keytabs'
        uid: 'root'
        gid: 'root' # was hadoop_group.name
        mode: 0o0755

## SPNEGO

Create the SPNEGO service principal in the form of "HTTP/{host}@{realm}" and place its
keytab inside "/etc/security/keytabs/spnego.service.keytab" with ownerships set to "hdfs:hadoop"
and permissions set to "0660". We had to give read/write permission to the group because the
same keytab file is for now shared between hdfs and yarn services.

      @call header: 'SPNEGO', ->
        @krb5.addprinc
          principal: options.spnego.principal
          randkey: true
          keytab: options.spnego.keytab
          uid: 'root'
          gid: options.hadoop_group.name
          mode: 0o660 # need rw access for hadoop and mapred users
        , options.krb5.admin

