# ansible-users

[![Build Status](https://travis-ci.com/chubchubsancho/ansible-users.svg?branch=master)](https://travis-ci.com/chubchubsancho/ansible-users)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?logo=github&style=flat)](https://raw.githubusercontent.com/chubchubsancho/ansible-users/master/LICENSE)

Role to manage users on a system.

## Role configuration

* `users_create_per_user_group` (default: true) - when creating users, also
  create a group with the same username and make that the user's primary
  group.
* `users_group` (default: users) - if users_create_per_user_group is _not_ set,
  then this is the primary group for all created users.
* `users_default_shell` (default: /bin/bash) - the default shell if none is
  specified for the user.
* `users_create_homedirs` (default: true) - create home directories for new
  users. Set this to false is you manage home directories separately.

## Creating users

Add a `users` variable containing the list of users to add. A good place to put
this is in `inventory/group_vars/users`.

Add a `environment_users` variable containing the list of users to add on 
specific inventory. Good place to put this is in `inventory/group_vars/groupname/users`.

Add a `host_users` variable containing the list of users to add on specific host.
A good place to put this is in `inventory/host_vars/hostname.

The following attributes are required for each user:

* username - The user's username.
* name - The full name of the user (gecos field)
* home - the home directory of the user to create (optional, defaults to /home/username)
* uid - The numeric user id for the user. This is required for uid consistency
  across systems.
* gid - The numeric group id for the group (optional). Otherwise, the
  uid will be used
* password - If a hash is provided then that will be used, but otherwise the
  account will be locked
* update_password - This can be either 'always' or 'on_create'
  * 'always' will update passwords if they differ. (default)
  * 'on_create' will only set the password for newly created users.
* group - optional primary group override
* groups - a list of supplementary groups for the user.
* ssh_key - This should be a list of ssh keys for the user. Each ssh key
  should be included directly and should have no newlines.
* generate_ssh_key - Whether to generate a SSH key for the user (optional, defaults to no).

In addition, the following items are optional for each user:

* shell - The user's shell. This defaults to /bin/bash. The default is
  configurable using the users_default_shell variable if you want to give all
  users the same shell, but it is different than /bin/bash.

## Creating groups

Add a `groups_to_create` variable containing the list of users to add. A good place to put
this is in `inventory/group_vars/group_name/users.

As for users creation those two variables exist for removing environment and host specific users
`environment_groups_to_create` and `host_groups_to_create`

The following attributes are required for each group:

* name - The group's name.
* gid - The group's id
* sudoers - Whether to add group on sudoers (optional, defaults to false).

## Deleting users

The `users_deleted` variable contains a list of users who should no longer be
in the system, and these will be removed on the next ansible run. The format
is the same as for users to add, but the only required field is `username`.
However, it is recommended that you also keep the `uid` field for reference so
that numeric user ids are not accidentally reused.

As for users creation those two variables exist for removing environment and host specific users
`environment_users_deleted` and `host_users_deleted`

## Examples

### global vars file (inventory/group_vars/all)

```yaml

    ---
    users:
      - username: foo
        name: Foo Barrington
        groups: ['wheel','systemd-journal']
        uid: 1001
        home: /local/home/foo
        ssh_key:
          - "ssh-rsa AAAAA.... foo@machine"
          - "ssh-rsa AAAAB.... foo2@machine"
    groups_to_create:
      - name: developers
        gid: 10000i
        sudoers: true
    users_deleted:
      - username: bar
        name: Bar User
        uid: 1002
```

### environment vars file (inventory/group_vars/group_name/users)

```yaml

    ---
    environment_users:
      - username: foo
        name: Foo Barrington
        groups: ['wheel','systemd-journal']
        uid: 1001
        home: /local/home/foo
        ssh_key:
          - "ssh-rsa AAAAA.... foo@machine"
          - "ssh-rsa AAAAB.... foo2@machine"
    environment_groups_to_create:
      - name: developers
        gid: 10000
    environment_users_deleted:
      - username: bar
        name: Bar User
        uid: 1002
```

### host vars file (inventory/host_vars)

```yaml

    ---
    host_users:
      - username: foo
        name: Foo Barrington
        groups: ['wheel','systemd-journal']
        uid: 1001
        home: /local/home/foo
        ssh_key:
          - "ssh-rsa AAAAA.... foo@machine"
          - "ssh-rsa AAAAB.... foo2@machine"
    host_groups_to_create:
      - name: developers
        gid: 10000
      - sudoers: true
    host_users_deleted:
      - username: bar
        name: Bar User
        uid: 1002
```
