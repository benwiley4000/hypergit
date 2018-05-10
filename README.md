# hypergit
Decentralized source code collaboration with git

### *✨ In Progress! ✨*

An app which mimics the integrated collaborative experience of GitHub, BitBucket, GitLab, etc.

The defining difference is that forks and merge requests (a.k.a. pull requests) work between repositories hosted on different servers, independent of any particular hosting service.

This means that different collaborators and forks hosted in different places can communicate with the ease of a conventional centralized service.

# TODO
1. Source code owners can host [Dat](https://github.com/datproject/dat) nodes which:
    * point to to a git repository available over https or ssh
   * contain a read-only (via Dat) history of issues, pull requests, code reviews, release notes, and other relevant meta information
2. Each node has an associated HTTPS server which accepts POST requests from users who would like to:
    * Create an issue or pull request
    * Comment in an issue or pull request
    * Close an issue or pull request (with appropriate permissions)
    * *Other standard git actions, such as merging a branch, may or may not be handled via this server, but aren't described here since they exist in native git*
3. Supports auth integration with apps like GitHub and BitBucket to enable the following:
    * Sync issues, pull requests, comments, release notes, etc. (the server node is responsible for this)
    * Facilitate selection of existsing repositories to share over hypergit
4. An Electron app with UI corresponding to everything described above
5. An easy mechanism for hosting a git repository directly with hypergit rather than using a third-party service.
