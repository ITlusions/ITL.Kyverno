# ITL.Kyverno

[Kyverno](https://kyverno.io/) for the ITL Talos cluster ([`ITlusions/ITL.K8s`](https://github.com/ITlusions/ITL.K8s)) - a Kubernetes-native policy engine (mutate/validate/generate admission control), introduced as **reusable cluster infrastructure**, not a one-off fix bolted onto a single app chart.

**Status: deployed and verified live** on the `itl` cluster, 2026-10-29. One policy, `spiffe-helper-nonroot`, is active and tested - see `docs/` for the full write-up, including two cluster-wide-admission-outage bugs found and fixed while getting it live (a `failurePolicy: Fail` default plus a JMESPath null-handling bug briefly blocked pod creation cluster-wide, twice, each caught within about a minute).

## Why this exists

[`ITL.Openproject.fork`](https://github.com/ITlusions/ITL.Openproject.fork) needed a non-root `securityContext` on the `spiffe-helper` sidecar container that [`ITL.Identity.Spire`](https://github.com/ITlusions/ITL.Identity.Spire)-based workload identity injects - without it, `spiffe-helper` (which defaults to root) writes `tls.key` as `0600 root:root`, unreadable by any application container running as a non-root uid. Patching this per-app, inside every chart that uses SPIFFE workload identity, does not scale - the next consumer hits the exact same bug and has to rediscover and re-fix it. A cluster-wide Kyverno `mutate` policy that patches any container named `spiffe-helper`, regardless of which Helm release or namespace it's in, fixes it once for every current and future consumer.

## This is a dependency wrapper, not hand-rolled manifests

Same reasoning as `ITL.Identity.Spire`: Kyverno's own admission-webhook/controller surface is large and security-sensitive, and upstream actively maintains it - re-implementing it by hand would just drift from upstream fixes. This chart tracks the **upstream `kyverno/kyverno` Helm chart as a real Helm dependency** (see `Chart.yaml`); `values.yaml` only overrides what's actually cluster-specific (currently just `admissionController.replicas`, since this is a single lab-scale cluster, not an HA requirement yet).

## Policies

- `spiffe-helper-nonroot` (`itl.kyverno/templates/spiffe-helper-nonroot.yaml`): the non-root mutate policy described above. See `ITL.Openproject.fork`'s docs (`docs/5-authentication/5.3-spiffe-mtls-postgres.md`) for the concrete bug this fixes, and `docs/2-policies/2.1-spiffe-helper-nonroot.md` in this repo for how it's implemented and the bugs found deploying it.
- `spiffe-helper-inject` (`itl.kyverno/templates/spiffe-helper-inject.yaml`): annotate a pod template with `spiffe.itlusions.local/inject: "true"` and it gets a full `spiffe-helper` native-sidecar setup (CSI volume, certs volume, config, container) at admission time - no chart changes needed. See `docs/2-policies/2.2-spiffe-helper-inject.md`.

## Deploy

```bash
cd itl.kyverno
helm dependency update
helm upgrade --install kyverno . -n kyverno --create-namespace
```

## Verify

```bash
kubectl get pods -n kyverno
kubectl get clusterpolicy
```

## Related repositories

- [`ITL.Identity.Spire`](https://github.com/ITlusions/ITL.Identity.Spire): the SPIRE server/agent/controller-manager this chart's first policy supports.
- [`ITL.Openproject.fork`](https://github.com/ITlusions/ITL.Openproject.fork): the consumer that surfaced the need for this.
- [`ITL.K8s`](https://github.com/ITlusions/ITL.K8s): the cluster documentation. This application is not listed in its index yet.
