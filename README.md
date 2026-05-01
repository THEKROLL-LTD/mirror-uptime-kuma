# mirror-uptime-kuma

Hardened container mirror of [louislam/uptime-kuma](https://github.com/louislam/uptime-kuma). THEKROLL's internal Uptime Kuma image, published publicly as a reference build.

## What this repository does

Each night, a GitHub Actions pipeline in this repository:

1. Checks `louislam/uptime-kuma` for a new release tag
2. If there is one, clones the upstream source, **rebuilds the entire Docker image chain from `node:22-bookworm-slim`** instead of pulling the opaque prebuilt `louislam/uptime-kuma:base2` family from Docker Hub
3. Detects upstream Dockerfile drift across all three of upstream's Dockerfiles (`docker/dockerfile`, `docker/debian-base.dockerfile`, `docker/builder-go.dockerfile`); blocks with an issue if anything changed since the last review
4. Scans the cloned source with Trivy (filesystem) and the built image with Trivy (container); produces SBOMs for both
5. If clean: pushes to `ghcr.io/thekroll-ltd/uptime-kuma` and opens a digest-pin PR
6. If findings: blocks the push, opens an issue, retains the full audit bundle for 90 days

## Image

`ghcr.io/thekroll-ltd/uptime-kuma:<tag>` and `ghcr.io/thekroll-ltd/uptime-kuma@sha256:<digest>`.

Tags track upstream Uptime Kuma releases. Digests are the authoritative pin.

## Differences from `louislam/uptime-kuma`

- **All bases self-built from `node:22-bookworm-slim`** (digest-pinned). Upstream's `:base2` and `:base2-slim` are prebuilt on Docker Hub by louislam's CI; we don't depend on them.
- **Go-healthcheck binary built from `golang:1.22-bookworm`** instead of upstream's EOL `golang:1-buster`.
- **Runs as `node` (UID 1000) by default** — equivalent to upstream's `:rootless` variant. Upstream's default `:release` runs as root; we don't replicate that.
- **No feature loss** — chromium, mariadb-server, ping, apprise, cloudflared, all fonts are present. Same monitor types work.
- **Single arch** (linux/amd64). Upstream ships armv7+arm64+amd64; we drop armv7 (cloudflared apt repo doesn't ship it) and currently build amd64 only.
- OCI labels populated.

The application code itself is byte-equivalent to upstream — no source patches.

## What we gain by self-building the base

- **Apt-layer CVE patch tempo**: a chromium fix lands in our image as soon as Debian publishes it (typically days), not when louislam bumps next (weeks).
- **One trust link removed**: we don't depend on louislam's Docker Hub account or CI infrastructure for the base layers.
- **Reproducibility**: every layer is built from a digest-pinned source by our pipeline, with audit logs and SBOM.

## What we explicitly do *not* fix

- **npm-layer CVEs in `package-lock.json`**: we mirror upstream's lockfile verbatim. CVEs in `protobufjs`, `fast-xml-parser`, `tar`, etc. are upstream's call to bump. We document our analysis in `.trivyignore` waivers but don't override versions — that would be a divergence from upstream behavior, which is out of scope for a mirror.

## No SLA

This is THEKROLL's own internal build, made public as a reference. There is no service-level agreement, no support commitment, no compatibility guarantee.

**For production-critical use**, fork the template and run your own pipeline: [THEKROLL-LTD/oss-mirror-build](https://github.com/THEKROLL-LTD/oss-mirror-build).

## License

The build system in this repository — workflow YAML, Dockerfile override, documentation — is licensed under Apache-2.0. See [`LICENSE`](LICENSE).

The container images produced here contain Uptime Kuma, which is licensed under MIT. The images inherit MIT. See [`NOTICE.md`](NOTICE.md).

## Related

- **Upstream:** [github.com/louislam/uptime-kuma](https://github.com/louislam/uptime-kuma) (MIT)
- **Template this repo was forked from:** [THEKROLL-LTD/oss-mirror-build](https://github.com/THEKROLL-LTD/oss-mirror-build) (Apache-2.0)
- **Sister mirrors:** [mirror-Gokapi](https://github.com/THEKROLL-LTD/mirror-Gokapi), [mirror-Plausible](https://github.com/THEKROLL-LTD/mirror-Plausible)

## Maintained by

[THEKROLL](https://thekroll.ltd) — DevOps consultancy from Cyprus. For production-critical use, don't depend on this mirror; fork the template and run your own pipeline at [`THEKROLL-LTD/oss-mirror-build`](https://github.com/THEKROLL-LTD/oss-mirror-build).
