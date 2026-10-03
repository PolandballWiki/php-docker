# PHP base images

Build PHP-FPM images with the extensions and system packages used by PolandballWiki.
PHP 8.1 and 8.3 retain their existing Dockerfiles. PHP 8.5 uses **8.5.11**;
its upstream multi-platform image is pinned by digest.

## PHP 8.5

The extension set is unchanged: calendar, intl, mbstring, mysqli, OPcache,
APCu, LuaSandbox and Redis. OPcache is built into PHP 8.5, so it is no longer
compiled separately. Existing PECL extensions are updated to APCu 5.1.28,
LuaSandbox 4.1.3 and Redis 6.3.0 for the new PHP ABI. Composer 2.10.3 is downloaded
at an exact version and checked against its recorded SHA256 before execution.

```sh
docker build -t ikigai-php-base:8.5.11 dockerfiles/php-base/8.5
docker run --rm ikigai-php-base:8.5.11 php -v
docker run --rm ikigai-php-base:8.5.11 php -m
```

Use `--network host` only if the local build network cannot resolve downloads.
Native AMD64 and ARM64 runners publish architecture digests. The merge job
combines **only the two digests belonging to the same PHP version**, then publishes
`8.5`, `8.5.11` and run-specific tags. `latest` follows only the newest supported
series (8.5); consumers should use a version plus immutable digest.

Pushes build changed Dockerfile directories. Manual dispatch builds all three
series; it does not combine different PHP versions into a single image. GitHub
Actions are pinned to reviewed commits. Publication uses the repository's
`GITHUB_TOKEN` with `packages: write`; no publishing PAT is required.

## Updating

Review the official PHP image digest, PECL compatibility and Composer checksum
before updating a Dockerfile. Changing the PHP ABI requires rebuilding all PHP
extensions and the consuming MediaWiki image. Preserve the chosen base digest in
MediaWiki's build manifest, regenerate its Composer lock if dependencies change,
and run application checks before promoting to another environment.

Alpine packages are fetched from the pinned base's package repositories, which
can receive updates: fixing PHP, PECL and Composer does not guarantee identical
image bytes across all future rebuilds. Registry digests identify the deployed
artifact. A package mirror/snapshot and signed release artifacts are possible
future improvements if stronger rebuild guarantees are required.
