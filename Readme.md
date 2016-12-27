# Role "Users"

Intended for managing a small number of Linux users in a simple and comfortable way.

## Variables

### users__initial_pass

Defines a password set for all new users (mainly for sudo).
Requires additionally `set_initial_pass: yes` for every user who should have a initial password.

Password change is enforced on first login - **users should be advised to log in to all servers asap!**

If variable is not set or empty string, no initial password is set.


### users__catalog

List of all possible users with account details.

Removed users may (and probably should) just stay in the list.
That also makes it easier to see which UIDs were already used.

Best use username as unique key for the dictionary;
using something different as key makes only sense when same username occurs more then once.


``` yml
# minimal user:

key:
  username: <username>
  uid: <uid>

user1:
  username: user1
  uid: 1001


# user with all possible settings:

user2:
  username: user2
  uid: 1001
  comment: "nice guy"           # (default: empty)
  groups: ['sudo','deploy']     # groups in addition to the primary group (default: [])
  get_root_email: yes           # user gets emails for user root (default: no)
  mail_forwards:                # writes ~/.forward to forward user mails to given addresses (default: [])
    - mail1@email.com
    - mail2@email.com
  set_initial_pass: yes         # sets the defined initial password and enforces change on login (default: no)
  ssh_keys:                     # a list of keys can be supplied (default: no key)
    - "ssh-rsa <here goes the key>"
  shell: /bin/csh               # (default: /bin/bash)

```


### users__present

Defines users to be present on the system.
List of keys from the `users__catalog`.

Example: `users__present: [user1, user2]`

**UID needs to be unique for each user** (and neither should UIDs from removed users be re-used).

**Removing a user from this list does not remove user from system!** See `users__removed`.


### users__removed

Defines users to be removed from system.
Could be removed from list after successful Ansible run, but might just as well be left there.

*Users' home dirs are preserved!*

Example: `users__removed: [user1, user2]`


---

## Howto

### Set a password later

`set_initial_pass` is only taken into account on first creation of user. If user is created without password at first, and later on he should have a password (let's say user got promoted to sudoers) this isn't possible straight away.

There is a quite easy workaround thought: remove and re-add  the user:

0. set `set_initial_pass: yes` for user in `users__catalog`
1. add user key to `users__removed`
2. run ansible -> user is removed, but his files stay.
3. remove user key from `users__removed` and make sure it's in `users__present`
4. run ansible -> user is created with initial password set
