
# Shinken Arbiter Start

    module.exports = header: 'Shinken Arbiter Start', handler: (options) ->
      options = options.options if options.options?

      @service.start name: 'shinken-arbiter'
