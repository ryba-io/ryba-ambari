
# Shinken Reactionner Stop

    module.exports = header: 'Shinken Reactionner Stop', handler: (options) ->
      options = options.options if options.options?

      @service.stop name: 'shinken-reactionner'

## Clean Logs

      @call header: 'Clean Logs', if: options.clean_logs, ->
        @system.execute
          cmd: 'rm /var/log/shinken/reactionnerd*'
          code_skipped: 1
