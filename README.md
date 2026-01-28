# What is Artifactory Remotes Really ?


## Claim:
The artifactory Docker remotes behaves as:

A stateful, content-transforming, origin-terminating gateway with an internal cache.

## 1. Artifactory terminates the client connection

From the client’s perspective, Artifactory is the registry:

> docker pull my-registry/myimage:tag

The client never sees:

the upstream registry hostname
upstream redirects
upstream auth challenges
upstream pagination
upstream referrers behavior

This is classic origin‑terminating proxy behavior.
RFC 7230 calls this a gateway.

# 2. Artifactory maintains a persistent, addressable cache

Unlike a transparent HTTP cache, Artifactory:

- stores manifests and blobs as real artifacts
- assigns them repo paths
- attaches properties
- indexes them
- exposes them to AQL
- uses them for virtual repository aggregation
- uses them for offline mode
- uses them for marker‑based blob resolution

This is not how an RFC‑compliant caching proxy behaves.
A transparent proxy caches responses, not artifacts.

# 3. Artifactory rewrites, transforms, and synthesizes content

Examples:

- It rewrites manifest paths (manifest.json vs list.manifest.json)
- It synthesizes marker files for foreign layers
- It rewrites blob URLs into internal repo paths
- It merges catalogs and tags across repos
- It enforces schema policies (e.g., block schema1)
- It rewrites headers (Docker-Content-Digest, ETag, etc.)

A transparent proxy must not do this.
Artifactory does it constantly.

This is content‑transforming gateway behavior.

# 4. Artifactory makes independent upstream decisions
A transparent proxy forwards the client’s request as-is.
Artifactory does not.

It may:
- fetch manifests by digest even if the client asked by tag
- fetch blobs using marker URLs unrelated to the client request
- retry upstream with different URLs
- fetch manifests or blobs proactively during indexing
- serve cached content even when upstream has newer content
- serve stale content when offline
- block content based on Xray/curation policies

This is not RFC caching behavior.

This is repository‑manager behavior.

# 5. Artifactory aggregates multiple upstreams (virtual repos)
A transparent proxy cannot merge:
- multiple registries
- multiple namespaces
- multiple tag lists
- multiple catalogs
- multiple referrers indexes

Artifactory does.

This is repository federation, not HTTP caching.

# 6. Artifactory maintains its own indexing system

RFC caches do not:
- build catalog indexes
- build tags indexes
- build referrers indexes
- maintain blob path caches
- maintain manifest metadata caches
- run background cleanup jobs
- run incremental indexers

Artifactory does.

This is artifact repository behavior, not proxy behavior.
