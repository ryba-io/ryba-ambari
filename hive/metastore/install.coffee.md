
# Hive Metastore Install

    module.exports =  header: 'Ambari Hive Metastore Install', handler: (options) ->

## Register

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

## SQL Connectors

      @call
        header: 'MySQL Client'
        if: options.db.engine in ['mariadb', 'mysql']
      , ->
        @service
          name: 'mysql'
        @service
          name: 'mysql-connector-java'
      @call
        header: 'Postgres Client'
        if: options.db.engine is 'postgresql'
      , ->
        @service
          name: 'postgresql'
        @service
          name: 'postgresql-jdbc'

## Metastore DB

      # @call header: 'Metastore DB', ->
      #   @db.user options.db, database: null,
      #     header: 'User'
      #     if: options.db.engine in ['mariadb', 'postgresql', 'mysql']
      #   @db.database options.db,
      #     header: 'Database'
      #     user: options.db.username
      #     if: options.db.engine in ['mariadb', 'postgresql', 'mysql']
      #   @db.schema options.db,
      #     header: 'Schema'
      #     if: options.db.engine is 'postgresql'
      #     schema: options.db.schema or options.db.database
      #     database: options.db.database
      #     owner: options.db.username
      @call header: 'DB Setup', ->
        switch options.db.engine
          when 'mariadb', 'mysql'
            # mysql_exec = "mysql -u#{options.db.admin_username} -p#{options.db.admin_password} -h#{options.db.host} -P#{options.db.port} "
            @system.execute
              cmd: db.cmd (merge {}, options.db, database: null) , """
              create database #{options.db.database};
              grant all privileges on #{options.db.database}.* to #{options.db.username}@'localhost' identified by '#{options.db.password}';
              grant all privileges on #{options.db.database}.* to #{options.db.username}@'%' identified by '#{options.db.password}';
              flush privileges;
              """
              unless_exec: db.cmd options.db, "use #{options.db.database}"

## Dependencies

    db = require 'nikita/lib/misc/db'
    {merge} = require 'nikita/lib/misc'
