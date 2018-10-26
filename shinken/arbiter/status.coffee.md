
# Shinken Arbiter Status

    module.exports = header: 'Shinken Arbiter Status', handler: (options) ->
      options = options.options if options.options?

      @service.status name: 'shinken-arbiter'
