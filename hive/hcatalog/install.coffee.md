
# Hive HCatalog Install

TODO: Implement lock for Hive Server2
http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH4/4.2.0/CDH4-Installation-Guide/cdh4ig_topic_18_5.html

    module.exports =  header: 'Ambari Hive HCatalog Install', handler: ({options}) ->

# Module Dependencies

    path = require 'path'
    db = require 'nikita/lib/misc/db'
    mkcmd = require 'ryba/lib/mkcmd'
