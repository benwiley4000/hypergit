# hypergit
Decentralized source code collaboration with git

### *✨ In Progress! ✨*

An app which mimics the integrated collaborative experience of GitHub, BitBucket, GitLab, etc.

The defining difference is that forks and merge requests (a.k.a. pull requests) work between repositories hosted on different servers, independent of any particular hosting service.

This means that different collaborators and forks hosted in different places can communicate with the ease of a conventional centralized service.

**Eventually, one option for hosting the git repository itself will be using Dat to share repository changes peer-to-peer!** However the initial goal is to sync discussions and pull requests peer-to-peer.

# TODO

The bullet points below might be sort of outdated; see [ARCHITECTURE.md](ARCHITECTURE.md) for a more formal description of the planned spec.

1. Source code owners can host [Dat](https://github.com/datproject/dat) nodes which:
    * point to to a git repository available over https or ssh
    * contain a read-only (via Dat) history of issues, pull requests, code reviews, release notes, and other relevant meta information
2. Each node runs a hyperdb feed (https://github.com/mafintosh/hyperdb) which is automatically authorized with write access and propagated across the network. This allows a user to:
    * Create an issue or pull request
    * Comment in an issue or pull request
    * Close an issue or pull request (with appropriate permissions, which are audited by other nodes in the network)
    * Remove a comment (with appropriate permissions)
    * Modify a user profile (with appropriate permissions)
    * Associate a user identity across multiple Dat nodes (by sending an identity confirmation request to another Dat address, and confirming on the other end)
3. Git action permissions are not handled by the Dat repository - they are handled by the repository itself. However there should be some way of associating a user account with git permissions in order to display relevant options (e.g. Merge Pull Request) in the UI.
4. Supports auth integration with apps like GitHub and BitBucket to enable the following:
    * Sync issues, pull requests, comments, release notes, etc. (the server node is responsible for this; supplementary apps e.g. for GitHub will be needed to create temporary branches and open pull requests)
    * Facilitate selection of existing repositories to share over hypergit
5. An Electron app with UI corresponding to everything described above
6. An easy mechanism for hosting a git repository directly with hypergit rather than using a third-party service.
