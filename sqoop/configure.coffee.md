
## Configuration

The module extends the "ryba/hadoop/core" module configuration.

*   `libs`, (array, string)
    List jar files (usually JDBC drivers) to upload into the Sqoop lib path.
    Use the space or comma charectere to separate the paths when the value is a
    string. This is for example used to add the Oracle JDBC driver "ojdbc6.jar"
    which cannt be downloaded for licensing reasons.
*   `user` (object|string)
    The Unix Sqoop login name or a user object (see Nikita User documentation).

Todo, with oozie, it seems like drivers must be stored in "/user/oozie/share/lib/sqoop".

Example:

```json
{
  "user": {
    "name": "sqoop", "system": true, "gid": "hadoop",
    "comment": "Sqoop User", "home": "/var/lib/sqoop"
  },
  "libs": "./path/to/ojdbc6.jar"
}
```

    module.exports = (service) ->
      {options} = service

## Identities

      # Group
      options.group ?= {}
      options.group = name: options.group if typeof options.group is 'string'
      options.group.name ?= 'sqoop'
      options.group.system ?= true
      # User
      options.user = name: options.user if typeof options.user is 'string'
      options.user ?= {}
      options.user.name ?= 'sqoop'
      options.user.system ?= true
      options.user.comment ?= 'Sqoop User'
      options.user.gid ?= options.group.name
      options.user.home ?= '/var/lib/sqoop'

## Environment

      # Layout
      options.conf_dir ?= '/etc/sqoop/conf'
      options.fqdn ?= service.node.fqdn

## Configuration

      options.configurations ?= {}
      # Env
      options.configurations['sqoop-env'] ?= {}
      options.configurations['sqoop-env']['content'] ?= "\n# Set Hadoop-specific environment variables here.\n\n#Set path to where bin/hadoop is available\n#Set path to where bin/hadoop is available\nexport HADOOP_HOME=${HADOOP_HOME:-{{hadoop_home}}}\n\n#set the path to where bin/hbase is available\nexport HBASE_HOME=${HBASE_HOME:-{{hbase_home}}}\n\n#Set the path to where bin/hive is available\nexport HIVE_HOME=${HIVE_HOME:-{{hive_home}}}\n\n#Set the path for where zookeper config dir is\nexport ZOOCFGDIR=${ZOOCFGDIR:-/etc/zookeeper/conf}\n\n# add libthrift in hive to sqoop class path first so hive imports work\nexport SQOOP_USER_CLASSPATH=\"`ls ${HIVE_HOME}/lib/libthrift-*.jar 2> /dev/null`:${SQOOP_USER_CLASSPATH}\""
      options.configurations['sqoop-env']['jdbc_drivers'] ?= ''
      options.configurations['sqoop-env']['sqoop.atlas.hook'] ?= 'false'
      options.configurations['sqoop-env']['sqoop_user'] ?= options.user.name
      # site
      options.sqoop_site ?= {}
      options.configurations['sqoop-site'] ?= {}
