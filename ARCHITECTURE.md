# architecture

This document describes the data model of hypergit and the enforcement of user permissions. The current layout is heavily influenced by GitHub, though everything is debatable and may be adjusted in future iterations.

### *Everything below is speculative since nothing has been implemented yet!* 😀

## data model

Hypergit is comprised of two types of entities: **Repositories** and **Users**. Each repository or user is represented by an individual [hyperdb](https://github.com/mafintosh/hyperdb) (a distributed multi-writer append-only log).

A hyperdb is made up of a geographically distant set of eventually-consistent logs (hypercores), which communicate as a swarm.

### a Repository:

* Is associated with a git repository, but the hyperdb does not technically contain the actual git repository's data. Rather, it contains:
  - A link to the git repository
  - Discussion threads for issues and pull requests. Each discussion comment points to a hash associated with a User
  - A list of hashes pointing to any repositories that have forked this one
  - A hash pointing to the forked repository, if this is a fork
  - A list of associations between a hypercore from this repository and a User entity
  - A list of associations between pairs of hypercores in this repository belonging to the same User
* Automatically authorizes any hypercore that joins the swarm to write to the hyperdb
* Relies on a set of application-level rules enforced across each client to avoid propagating writes that violate user permissions

### a User:

* Is associated with a person
* May be writable from multiple machines, but unlike a repository, authorization must be explicit
* May be associated with many repositories
* May be associated with many authorized hypercores for the same repository (since a user may connect to the same repository from many machines)
* Contains:
  - Descriptive and contact information
  - A list of hashes pointing to associated hypercores in repositories

## permissions

This describes permissions pertaining to interactions with repository metadata, ***not*** interactions with the git repository itself.

**TODO: we should define how the UI knows whether to display UI elements derivative of git repository permissions, i.e.:**
  - The merge button in a pull request
  - The button to create a new branch
  - The button to edit a text file from the UI
  - The button to add a tag or release notes

### User roles in a repository:

These roles may be, but are not strictly, tied to permissions on the git repository itself.

* A *normal user* is able to freely create and comment in issues and pull requests, close their own discussion threads, and create forks of the repository
* A *collaborator* has the permissions of a normal user, plus the ability to mute and un-mute normal users and other collaborators, open and close arbitrary threads, and lock threads to collaborators and admins.
* An *admin* has the permissions of a collaborator, plus the ability to freely adjust roles of other users, and to mute and un-mute admins (including themselves).

#### User status:
* A *muted user* is prevented from participating in issues and pull requests, either persistently or on a timed basis.

### Prohibited behaviors

A hypercore will be ignored by its peers if it commits any of these prohibited behaviors:
* Forge a comment pointing to the wrong user (assuming the commenting hypercore is known to be associated with a different user)
* Attempt an action in violation of the user role and status which are in effect for that user at the time of the action
- Some prohibited actions shouldn't result in blacklisting the peer. For instance, if a user has been muted by an admin but that change doesn't propagate to the user's hypercore until after they have sent a comment, the comment should be hidden, and the user's hypercore should remain connected to the swarm.

If a user attempts to associate their hypercore with another hypercore confirmed as belonging to a different user, the user should receive a warning, and pair confirmation from the other end should be prevented from taking effect. If this is attempted many times, the hypercore should be ignored by peers.

Client business logic should take special care to avoid committing violations, in defense against case the UI becomes out-of-sync with the local hyperdb.

### Application of user roles

Although roles are spoken about as associated with a User, they are actually applied to the first hypercore in a repository known to be associated with the selected user. Later, any hypercore associated with that user will use the same permissions.

### Connection of user hypercores

When a user connects to a repository from a new machine, identity is confirmed by forming two double connections:
* The User must update to reference the new hypercore in the repository hyperdb, and an association to the User must be committed by the hypercore in the repository
* If the User is already associated with another hypercore in the repository, one of the already-associated hypercores must commit an association with the new hypercore.
  - In the odd case that a user associates with a repository for the first time from two different machines before the hypercores have had time to sync, there will be no initial warning, but the second-linked hypercore should become temporarily unusable until after confirmation once replication has happened. Committed actions should be temporarily nullified (for the second hypercore).

It should be noted that the User-repository connection is used to confirm identity in the UI (a user is tentatively listed as "maybe" someone until confirmation has occurred).

The hypercore connection across the repository is used as the only source of truth for permissions. This may make recovery of old repositories somewhat inconvenient for users, but keeping permissions confined to the append-only log of the repository data means that we can safely avoid cases where we accidentally blacklist a peer for violating permissions when the User data becomes out-of-sync with the repository.

