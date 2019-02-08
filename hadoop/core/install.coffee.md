
# Hadoop Core Install

    module.exports = header: 'Hadoop Core Install', handler: ({options}) ->

## Registry

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'


## Packages

Install the "hadoop-client" and "openssl" packages as well as their
dependecies.

The environment script "hadoop-env.sh" from the HDP companion files is also
uploaded when the package is first installed or upgraded. Be careful, the
original file will be overwritten with and user modifications. A copy will be
made available in the same directory after any modification.

      @call header: 'Packages', ->
        @service
          name: 'openssl-devel'

## Web UI

This action follow the ["Authentication for Hadoop HTTP web-consoles"
recommendations](http://hadoop.apache.org/docs/r1.2.1/HttpAuthentication.html).

      @system.execute
        header: 'WebUI'
        cmd: 'dd if=/dev/urandom of=/etc/hadoop/hadoop-http-auth-signature-secret bs=1024 count=1'
        unless_exists: '/etc/hadoop/hadoop-http-auth-signature-secret'
