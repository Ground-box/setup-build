# setup-build

Build Docker images on a Groundbox builder: BuildKit on our box, 8 vCPUs, with a layer cache kept per project between builds. `docker/build-push-action` works unchanged.

```yaml
- uses: Ground-box/setup-build@v1
  with:
    api-key: ${{ secrets.GROUNDBOX_API_KEY }}
- uses: docker/build-push-action@v6
  with:
    push: true
    tags: ghcr.io/acme/app:latest
```

- `api-key`: a Groundbox API key with `builds:write` (Keys page in the dashboard). Remote builds need the paid plan.
- `project`: names the cache (default: the repository). Builds of one project share it; teams never share.
- `use`: make the builder the default (default `true`). The output `name` is the builder's name, for `builder:` on build-push-action.

Works on any runner with Docker and buildx: Groundbox runners, GitHub-hosted runners, your own. The step starts the builder (up to a minute when it was stopped), then runs a small proxy for the rest of the job: buildx's remote driver talks to a local socket, and each connection goes to the builder over the Groundbox API (HTTPS WebSocket). The builder is not on the internet.

The result of a build stays in the builder's cache unless you push it (`push: true`) or load it (`load: true`, which sends the image back to the runner).

Images are built natively for `linux/amd64`. `linux/arm64` builds run under QEMU emulation: they work, slowly.

`dist/proxy.mjs` is a bundle of the Groundbox CLI's build proxy and `ws`, built from the Groundbox repository.
