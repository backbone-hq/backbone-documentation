---
hide:
- navigation
---

## Motivation 

!!! Note "Alpha-grade software"
    Backbone is being battle-tested and **should not be used in production**. If you deploy anything using Backbone, kittens will very likely die. You should assume that all data uploaded could disappear at any time, for any reason, and especially when you need it. This makes Backbone the perfect place to test out novel Error Correction Code techniques.
    If you're feeling adventurous, feel free to join our [:material-slack: Slack Community](https://join.slack.com/t/backbone-dev/shared_invite/zt-rx3gxw5h-87SNvKVte8lEZN3GeNe4vw) to provide feedback and help improve Backbone!

Security experts often discuss the tradeoffs between security and usability. 
We believe this is a false dichotomy; you can achieve both without having to relinquish the tangible benefits of cloud infrastructure.
We therefore present Backbone, an end-to-end encrypted key-value store.

Backbone's aim is to ensure that the only entity you have to trust is yourself. 
You should not have to trust your cloud provider to protect your data - whether from foreign governments, cyber criminals or themselves.

Backbone makes it possible to securely use third-party hardware for storage with cryptographically-guaranteed granular access control.
We believe embedding end-to-end cryptography into your applications is the ideal way to securely store, manage and share mission-critical secrets.

## Overview

Backbone is an enriched key-value store with enterprise capabilities designed for maximum security. We could get thoroughly pwned [XKCD #538](https://xkcd.com/538/) style and, so long as you keep a few pieces of information secure, our adversaries would be able to recover none of your data (unless they've broken ECC, in which case you have bigger problems to worry about).
At the most fundamental level, Backbone is a collection of `workspaces`. 

Workspaces in turn are collections of `users` and act as the largest units of control and isolation in Backbone; all actions you perform occur inside a `workspace` and there are no cross-workspace interaction of any kind (until we find a way to monetise them).
Workspaces are controlled by a subset of their users: those with `root` permissions. Don't worry about permissions for now, ~~it's not like you'll ever bother to drop your root permissions~~ we'll explain them later. The key point to remember is that `workspaces` consist of `users` some of whom can manage the workspace and invite other users to it.

Once you've got your workspace and your users set up, you probably feel like an ~~idiot~~ excited prospective customer because you can't do anything with them. But fear not, Backbone has an exceptionally powerful key-value `store` that you can use as a marginally better alternative to pastebin.
The store consists of two types of objects: `entries` and `namespaces`. You can think of entries as files in a filesystem and namespaces are directories. We motivate you to be efficient with your storage by limiting the maximum size of your "files" to 2,048 bytes. You can thank us later.
Once you've created some entries and namespaces, you probably want to share them with your friends so they can access your email address, bank account and Dunkin Donuts subscription. 

With Backbone, you can destroy your life in this way with a single command and no confirmation prompts.
The best part is that, because we're in alpha, you don't even have to pay us for the privilege, that comes later. 

Rest assured that when we ~~achieve massive scale~~ increase user counts to lower costs, we will ~~jack up our prices to the moon~~ extend generous offers to our early subscribers, like true capitalists.

### Installation

The best way to get your hands on Backbone is to install the `backbone` package with your favourite Python package manager - unless it's conda.

This will install both the client library as well as the `backbone` command-line interface.

=== "Python: pip"
    ```
    pip install backbone
    ```
=== "Python: poetry"
    ```
    poetry add backbone
    ```
=== "Python: conda"
    ```
    conda install anaconda-clean && anaconda-clean --yes
    $(command -v chrome firefox) https://python-poetry.org/docs/#installation
    ```

### Initialization

The backbone ecosystem revolves around the concept of workspaces. Your applications, friends and users and their pets will all be part of your workspace.

!!! Note ""
    The user you create during initialization will automatically be granted `ROOT` permissions; Every workspace must always have at least one `ROOT` user.

#### Private Keys

=== "Python"
    ```python
    from backbone.sync import BackboneClient

    with BackboneClient(workspace, username, private_key) as client:
        client.workspace.create(display_name)
    ```
=== "CLI"
    ```bash
    backbone workspace create $WORKSPACE $DISPLAY_NAME $USERNAME
    ```

#### Passwords

Backbone supports using passwords for convenience. If you intend to use a password instead of a private key, we strongly recommend that its entropy is sufficient under your threat model.
Using passwords intentionally incurs a performance hit to discourage brute-force attacks.

=== "Python"
    ```python
    from backbone.sync import BackboneClient
    
    with BackboneClient.from_credentials(workspace, username, password) as client:
        client.workspace.create(display_name)
    ```
=== "CLI"
    ```bash
    backbone workspace create $WORKSPACE $DISPLAY_NAME $USERNAME --password
    ```

!!! Note ""
    You can use the `--password` flag to use passwords instead of private keys. Note that this intentionally incurs a performance hit.

### Authentication

Now that your workspace exists, you'll have to authenticate to be able to perform actions within your workspace.
This is the process of proving that you own your private key or password. If successful, Backbone will give you an ephemeral token that lasts 24 hours by default.

=== "Python"
    ```python
    client.authenticate()
    ```
=== "CLI"
    ```bash
    backbone authenticate workspace username
    ```

### Storage

Secure storage is the heart of your backbone - though that's certainly not anatomically correct. The store is a tree; whereby the root namespace (with a prefix of ``) is the de-facto owner of all child resources.
Users can then be granted access to a resource and inherit access to resources owned by their namespaces via a `chain`.

#### Accessing entries

=== "Python"
    ```python
    client.entry.get(key)
    ```
=== "CLI"
    ```bash
    backbone entry get $KEY
    ```

#### Setting entries

=== "Python"
    ```python
    client.entry.set(key, value)
    ```
=== "CLI"
    ```bash
    backbone entry set $KEY $VALUE
    ```

#### Deleting entries

=== "Python"
    ```python
    client.entry.delete(key)
    ```
=== "CLI"
    ```bash
    backbone entry delete $KEY
    ```

#### Creating namespaces

=== "Python"
    ```python
    client.namespace.create(key)
    ```
=== "CLI"
    ```bash
    backbone namespace create $KEY
    ```

#### Deleting namespaces

=== "Python"
    ```python
    client.namespace.delete(key)
    ```
=== "CLI"
    ```bash
    backbone namespace delete $KEY
    ```

### User Management

Your `workspace` can be an awfully lonely place when you create your first account. Luckily we have a solution for you, and you'll only have to sell one of your kidneys! You can invite additional users to your workspace with different sets of permissions so you can feel smug about your `root` access.

#### User invitation

Inviting users is a two-step process. The user first has to generate their key (and user payload) that an administrator then imports into the workspace.

##### Generating a user payload

=== "CLI"
    ```bash
    backbone user generate $USERNAME    
    ```

##### Importing users

=== "Python"
    ```python
    backbone.user.create(username, public_key)
    ```
=== "CLI"
    ```bash
    backbone user import $PAYLOAD
    ```

### Data Access Control

We're told sharing is caring. Just make sure not to invite the fox into the henhouse.

Access control within Backbone is separated into a variety of types:

| Access | Description |
| ------ | ----------- |
| READ   | Ability to read a resource's value |
| WRITE  | Ability to create or modify a resource |
| DELETE | Ability to remove a resource |
| EJECT  | Ability to interrupt the inheritance of grants for a namespace |


!!! Note "Endpoint control"
    Only users with the `STORE_SHARE` permission can share entries and namespace with other users

!!! Note ""
    The interfaces for granting and revoking access are equivalent for entries and namespaces

#### Grants

=== "Python"
    ```python
    client.entry.grant(key, username)
    ```
=== "CLI"
    ```bash
    backbone entry grant $KEY $USERNAME
    ```

#### Revocation

=== "Python"
    ```python
    client.entry.revoke(key, username)
    ```
=== "CLI"
    ```bash
    backbone entry revoke $KEY $USERNAME
    ```

### Endpoint Access Control

This form of access control revolves around limiting a user's access to a particular set of related endpoints.

| Permission  | Description |
| ----------- | ----------- |
| ROOT        | Access override to all endpoints |
| USER_MANAGE | Access to create, delete and import users |
| STORE_USE   | Access to the backbone store |
| STORE_SHARE | Ability to share access to the backbone store |

### Isolated Namespaces

Isolation is a mechanism to interrupt the inheritance of grants from higher namespaces. It requires the ownership of `EJECT` access to the higher namespace and will result in a namespace where the initial creator is the sole owner of that namespace.

=== "Python"
    ```python
    client.namespace.create(key, isolated=True)
    ```
=== "CLI"
    ```bash
    backbone namespace create $KEY --isolated
    ```


#### Frequently Asked Questions

> I forgot my password, can you help me recover it?

This is a significant improvement to your security posture. You are no longer susceptible to [XKCD #538](https://xkcd.com/538/).
While Backbone is in alpha, this is not an issue; you will not be charged or prosecuted.

> I don't like your service, can I speak to the manager?

Yes, please reach out to devnull@backbone.dev and we'll be happy to discard your message.
