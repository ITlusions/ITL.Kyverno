# ITL.Kyverno documentation index

Documents are grouped into numbered folders. Each file is named `x.y-title.md`, where `x` is the folder number. Start with **1. Overview**.

State on 2026-10-29: deployed and verified live on the `itl` cluster (Helm release `kyverno`, revision 7). One policy (`spiffe-helper-nonroot`) is `Ready` and tested against both a matching and a non-matching pod - see [1.2 Deployment](1-overview/1.2-deployment.md) for the two cluster-wide-outage bugs found and fixed getting there.

## 1. Overview

- [1.1 Architecture](1-overview/1.1-architecture.md): what this wrapper is, why Kyverno, the dependency-wrapper pattern, mutate vs. validate vs. generate.
- [1.2 Deployment](1-overview/1.2-deployment.md): install steps, current state, verification.

## 2. Policies

- [2.1 SPIFFE Helper non-root](2-policies/2.1-spiffe-helper-nonroot.md): the first and so-far-only policy - why it exists, what it patches, the bug it fixes, its limits.

## Related repositories

- [`ITL.Identity.Spire`](https://github.com/ITlusions/ITL.Identity.Spire): the SPIRE deployment this chart's first policy supports.
- [`ITL.Openproject.fork`](https://github.com/ITlusions/ITL.Openproject.fork): the consumer that surfaced the need for this repo - see its own `docs/5-authentication/5.3-spiffe-mtls-postgres.md` for the live incident this policy would have prevented.
- [`ITL.K8s`](https://github.com/ITlusions/ITL.K8s): the cluster documentation. This application is listed in its index.
