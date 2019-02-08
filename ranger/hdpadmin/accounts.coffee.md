
# Ranger Admin Setup

    module.exports =  header: 'Ambari Ranger Admin Setup', handler: ({options}) ->

## Register

      @registry.register 'ranger_user', 'ryba/ranger/actions/ranger_user'


## User Accounts
Deploying some user accounts. This middleware is here to serve
as an example of adding a user,and giving it some permission.
Requires `admin` user to have `ROLE_SYS_ADMIN`.
Method to check is user account already exit is not identical base on user source.
Indeed usersource to 1 means external user and so unknown password.

      @ranger_user (
        header: "Account #{name}"
        username: options.admin.username
        password: options.admin.password
        url: options.install['policymgr_external_url']
        user: user
      ) for name, user of options.users
