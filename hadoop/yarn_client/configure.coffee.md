

# YARN Client Configure

    module.exports = (service) ->
      options = service.options
      options.configurations ?= {}

## Environment

      options.log_dir ?= '/var/log/hadoop-yarn'
      # options.pid_dir ?= '/var/run/hadoop-yarn'
      options.conf_dir ?= service.deps.hadoop_core.options.conf_dir
      options.opts ?= ''
      options.heapsize ?= '1024'
      options.home ?= '/usr/hdp/current/hadoop-yarn-client'
      # Misc
      options.java_home ?= service.deps.java.options.java_home

## Identities

      options.group = merge {}, service.deps.hadoop_core.options.yarn.group, options.group
      options.user = merge {}, service.deps.hadoop_core.options.yarn.user, options.user
      # Kerberos Test Principal
      options.test_krb5_user ?= service.deps.test_user.options.krb5.user
      options.fqdn ?= service.node.fqdn

## Configuration

      options.yarn_site ?= {}

## Wait

      options.wait_yarn_ts = service.deps.yarn_ts[0].options.wait
      options.wait_yarn_rm = service.deps.yarn_rm[0].options.wait

## Dependencies

    {merge} = require 'nikita/lib/misc'
