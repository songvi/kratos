---
id: index
title: Hooks
---

Hooks execute logic before or after a flow (login, registration, settings, ...):

- *Before login:* is executed when a login is initiated.
- *After login:* is executed after a login was successful.
- *Before registration:* is executed when a registration is initiated.
- *After registration:* is executed when a registration was successful:
  - *Before persisting:* runs before the identity is saved in the database.
  - *After persisting:* runs after the identity was saved in the database.
- *Before settings:* is executed when a settings is initiated.
- *After settings:* is executed when a settings was successful:
  - *Before persisting:* runs before the identity is saved in the database.
  - *After persisting:* runs after the identity was saved in the database.

## Login

Hooks running before & after successful user authentication are defined per
Self-Service Login Strategy in ORY Kratos' configuration file.

### Before

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  login:
    before:
      - hook: redirect
        config:
          to: https://www.ory.sh/maintenance
```

#### `redirect`

The `redirect` job sends HTTP 302 Found and redirects the client
to the specified endpoint. This is useful
when you want to disable any settings functionality (e.g. due to maintenance).

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  login:
    before:
      - hook: redirect
        config:
          to: https://www.ory.sh/maintenance
```


### After

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  login:
    after:
      oidc:
        - hook: redirect
          config:
            to: https://www.ory.sh/
      password:
        - hook: revoke_active_sessions
```

#### `redirect`

The `redirect` job sends HTTP 302 Found and redirects the client
to the specified endpoint. This hook overrides the default redirection
behaviour and enforces the specified redirect URL.

Using this hook should be an exception.

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  login:
    after:
      <strategy>:
        - hook: redirect
          config:
            to: https://url-to-redirect/to
```

#### `revoke_active_sessions`

The `revoke_active_sessions` will delete all active sessions for that user on
successful login:

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  login:
    after:
      <strategy>:
        - hook: revoke_active_sessions
          # can not be configured
```

## Registration

Hooks running before & after successful user registration are defined per
Self-Service Registration Strategy in ORY Kratos' configuration file.

### Before

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  registration:
    before:
    - hook: redirect
      config:
        to: https://www.ory.sh/maintenance
```

#### `redirect`

The `redirect` job sends HTTP 302 Found and redirects the client
to the specified endpoint. This is useful
when you want to disable any settings functionality (e.g. due to maintenance).

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  registration:
    before:
    - hook: redirect
      config:
        to: https://www.ory.sh/maintenance
```

### After

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  registration:
    after:
      oidc:
        - hook: session
      password:
        - hook: session
```

#### `session`

Adding the `session` hook signs the user immediately in once the account has been created.
It runs after the identity has been saved to the database.

::: warn
Using this job as part of your post-registration workflow makes your system
vulnerable to
[Account Enumeration Attacks](../../concepts/security.md#account-enumeration-attacks)
because a threat agent can distinguish between existing and non-existing
accounts by checking if `Set-Cookie` was sent as part of the registration
response.
:::

It sends  a `Set-Cookie` header which contains the session
cookie. To use it, you must first define one or more (for secret rotation)
session secrets and then use it in one of the `after` work flows:

```yaml title="path/to/my/kratos.config.yml"
secrets:
  session:
    - something-super-secret # The first entry will be used to sign and verify session cookies

    # All other entries will be used to verify session cookies that were signed before "something-super-secret" became
    # the current signing secret.
    - old-session-secret
    - older-session-secret
    - ancient-session-secret

selfservice:
  registration:
    after:
      <strategy>:
        - hook: session
          # can not be configured
```

#### `redirect`

The `redirect` hook sends HTTP 302 Found and redirects the client
to the specified endpoint.

::: Note
Using this hook for registration disables user registration because it runs
before the identity is saved to the database. It may
be useful in cases where you temporary suspend user registration.
:::

Using this hook should be an exception.

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  registration:
    after:
      <strategy>:
        - hook: redirect
          config:
            to: https://url-to-redirect/to
```

### `verify`

The `verify` hook checks for verifiable email addresses and sends a verification / activation
email. For more information,
please read [User Verification and Account Activation](../flows/verify-email-account-activation.mdx).

## Settings

Hooks running before & after successfully updating user settings and are defined per
Self-Service Settings Strategy in ORY Kratos' configuration file.

### Before

Settings flows do not have before hooks.

### After

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  settings:
    after:
      - hook: redirect
        config:
         to: https://www.ory.sh/
```

#### `redirect`

The `redirect` job sends HTTP 302 Found and redirects the client
to the specified endpoint.

Per default, the settings endpoint returns to the settings page
with the original settings request ID. This is useful
when you want to show e.g. a success message indicating
that the data has successfully been saved.

To override this behaviour, use this redirect hook.

```yaml title="path/to/my/kratos.config.yml"
selfservice:
  settings:
    after:
      <strategy>:
        - hook: redirect
          config:
            to: https://www.ory.sh/settings-updated
```
