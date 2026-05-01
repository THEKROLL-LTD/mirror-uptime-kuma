# NOTICE

## Repository license

The build system in this repository — `Dockerfile.override`, GitHub Actions
workflow, configuration files, and documentation — is licensed under
Apache-2.0. See `LICENSE` for the full text.

No Uptime Kuma source code is contained in this repository. Upstream source
is cloned fresh at build time and discarded after the image is produced.

## Image license

The container images produced by this repository and published to
`ghcr.io/thekroll-ltd/uptime-kuma` contain a compiled build of Uptime Kuma,
which is licensed under MIT. The images are therefore distributed under the
MIT license, inheriting upstream's terms.

- Upstream: https://github.com/louislam/uptime-kuma
- Upstream license: https://github.com/louislam/uptime-kuma/blob/master/LICENSE

The mirror build adds no new code to the application — `Dockerfile.override`
only changes the base image chain (replaces opaque prebuilt bases with a
direct `node:22-bookworm-slim` build) and rebuilds the Go-healthcheck binary
from a current Go version. No Uptime Kuma source is patched.

## No SLA

This mirror is THEKROLL's internal build, published publicly as reference.
No service-level agreement, no support commitment, no compatibility guarantee.
For production-critical deployments, fork the template at
`github.com/THEKROLL-LTD/oss-mirror-build` and run your own pipeline.
