# Chainguard Images

Chainguard Images is a collection of container images designed for minimalism and security.

Many of these images are _distroless_; they contain only an application and its runtime
dependencies. There is no shell or package manager.

All these images are built using [apko](https://github.com/chainguard-dev/apko) and
[melange](https://github.com/chainguard-dev/melange). Combined, these tools provide for a
reproducible, declarative approach to building OCI images. We aim to build images from the latest
sources every night, ensuring that vulnerabilities are addressed as quickly as possible.

melange lets you build apks using declarative YAML pipelines. apks are .apk packages compatible with
the package manager used by Alpine, similar to .deb or .rpm for instance.

apko lets you bundle a collection of APKs into an OCI image using a declarative YAML manifest.

Chainguard Images provide SBOM support and signatures for known provenance and more secure base
images. They can be part of an approach to a secure software factory.

Many of the images use the [Wolfi undistro](https://github.com/wolfi-dev/), and we are working on
porting the rest.

## Find and use Chainguard Images

Our images are available via cgr.dev.

For example, to pull the apko image with Docker:

```
docker pull cgr.dev/chainguard/apko 
```

There are various types of images available. The full list can be found in the [Release Status
section](https://github.com/chainguard-images#release-status) or you can use the repository search
function. Some categories and highlights follow.

### Minimal Base Images

The static image is designed for running statically compiled binaries. Unlike the completely empty
scratch image, we do include some commonly required files and directories such as:

 - Root certificate data
 - `/etc/passwd` and `/tmp`

The most common way to use this image is with a multistage Dockerfile. For example:

```
# syntax=docker/dockerfile:1.4
FROM cgr.dev/chainguard/gcc-musl:latest as build

COPY <<EOF /hello.c
#include <stdio.h>
int main() { printf("Hello World!"); }
EOF
RUN cc -static /hello.c -o /hello

FROM cgr.dev/chainguard/static:latest

COPY --from=build /hello /hello
CMD ["/hello"]
```

We can compile and run this with:

```
$ docker build -t c-static .
...
$ docker run c-static
Hello World!
```

In some cases it is easier to produce binaries that are dynamically linked against glibc or musl,
but otherwise self-contained. In those cases you can use the following images:

 - [glibc-dynamic](https://github.com/chainguard-images/glibc-dynamic)
 - [musl-dynamic](https://github.com/chainguard-images/musl-dynamic)

### Language Runtimes

These are still largely in progress, but we are working on images for runtimes such as the JRE.

### Compilers

These images can be used to compile artifacts for use in runtime or base images. For instance the go
image can be used to compile golang code for use in the static base image.

We have plans to include extra tooling in these images for creating provenance information such
as SBOMs.

Examples:

 - [go](https://github.com/chainguard-images/go)
 - [gcc-glibc](https://github.com/chainguard-images/gcc-glibc)
 - [gcc-musl](https://github.com/chainguard-images/gcc-musl)

### Middleware and Applications

These images contain stand-alone software, such as databases and web servers, as well as tooling
such as apko and melange.

Because our images are constantly rebuilt with the latest sources and include the absolute minimum
of dependencies, they typically have significantly less vulnerabilities than equivalent images.

For example:
 - [nginx](https://github.com/chainguard-images/nginx) 
 - [git](https://github.com/chainguard-images/git)
 - [apko](https://github.com/chainguard-images/apko)
 - [melange](https://github.com/chainguard-images/melange)

## Signatures

All Chainguard Images are signed using [Sigstore](https://www.sigstore.dev/), and you can check
the signature using [`cosign`](https://docs.sigstore.dev/cosign/overview). For our apko image example,
you can run the following:

```
COSIGN_EXPERIMENTAL=1 cosign verify cgr.dev/chainguard/apko | jq 
```

Your output will indicate that the cosign claims were validated.

## SBOMs

All Chainguard Images come with a [Software Bill Of Materials
(SBOM)](https://www.chainguard.dev/unchained/what-an-sbom-can-do-for-you) generated at build-time.
The SBOM can be downloaded using the [`cosign`](https://docs.sigstore.dev/cosign/overview) tool e.g:

```
$ cosign download sbom --platform linux/amd64 cgr.dev/chainguard/nginx | jq
WARNING: Downloading SBOMs this way does not ensure its authenticity. If you want to ensure a tamper-proof SBOM, download it using 'cosign download attestation <image uri>' or verify its signature.
Found SBOM of media type: spdx+json
{
  "SPDXID": "SPDXRef-DOCUMENT",
  "name": "sbom-sha256:05f301c5da6b701a1024dce52ccb9c58a61f98cd9816432ace5c0f7bfef40df7",
  "spdxVersion": "SPDX-2.3",
  "creationInfo": {
    "created": "2022-09-22T01:41:52Z",
    "creators": [
      "Tool: apko (canary)",
      "Organization: Chainguard, Inc"
    ],
    "licenseListVersion": "3.16"
  },
  "dataLicense": "CC0-1.0",
  "documentNamespace": "https://spdx.org/spdxdocs/apko/",
  "documentDescribes": [
    "SPDXRef-Package-sha256-a8aa25c0811fec7afc8de07be49340e27c4bb752fde24a1dae0509d1a3824b3a"
  ],
  "packages": [
  ...
```

Make sure to specify the required platform, or the resultant SBOM may only cover the image manifest and not
the contents of the image.

## Learn more 

You can learn more about Wolfi, distroless images, apko, and melange from the following
articles:

 - [Introducing apko: bringing distroless nirvana to Alpine Linux](https://blog.chainguard.dev/introducing-apko-bringing-distroless-nirvana-to-alpine-linux/)
 - [Minimal Container Images: Towards a More Secure Future](https://blog.chainguard.dev/minimal-container-images-towards-a-more-secure-future/)
 - [Secure Your Software Factory with melange and apko Media](https://blog.chainguard.dev/secure-your-software-factory-with-melange-and-apko/)

## Support

Support is provided on a best-effort basis via GitHub issues.

For commercial support, please see [Chainguard](https://www.chainguard.dev/chainguard-images).

## Release Status

| Repo  |  Status |
| ----- | ----- |
| [alpine-base](https://github.com/chainguard-images/alpine-base) | [![CI status](https://github.com/chainguard-images/alpine-base/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/alpine-base/actions/workflows/release.yaml) |
| [apko](https://github.com/chainguard-images/apko) | [![CI status](https://github.com/chainguard-images/apko/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/apko/actions/workflows/release.yaml) |
| [busybox](https://github.com/chainguard-images/busybox) | [![CI status](https://github.com/chainguard-images/busybox/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/busybox/actions/workflows/release.yaml) |
| [gcc-glibc](https://github.com/chainguard-images/gcc-glibc) | [![CI status](https://github.com/chainguard-images/gcc-glibc/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/gcc-glibc/actions/workflows/release.yaml) |
| [gcc-musl](https://github.com/chainguard-images/gcc-musl) | [![CI status](https://github.com/chainguard-images/gcc-musl/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/gcc-musl/actions/workflows/release.yaml) |
| [git](https://github.com/chainguard-images/git) | [![CI status](https://github.com/chainguard-images/git/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/git/actions/workflows/release.yaml) |
| [glibc-dynamic](https://github.com/chainguard-images/glibc-dynamic) | [![CI status](https://github.com/chainguard-images/glibc-dynamic/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/glibc-dynamic/actions/workflows/release.yaml) |
| [go](https://github.com/chainguard-images/go) | [![CI status](https://github.com/chainguard-images/go/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/go/actions/workflows/release.yaml) |
| [hello-world](https://github.com/chainguard-images/hello-world) | [![CI status](https://github.com/chainguard-images/hello-world/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/hello-world/actions/workflows/release.yaml) |
| [jdk](https://github.com/chainguard-images/jdk) | [![CI status](https://github.com/chainguard-images/jdk/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/jdk/actions/workflows/release.yaml) |
| [ko](https://github.com/chainguard-images/ko) | [![CI status](https://github.com/chainguard-images/ko/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/ko/actions/workflows/release.yaml) |
| [melange](https://github.com/chainguard-images/melange) | [![CI status](https://github.com/chainguard-images/melange/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/melange/actions/workflows/release.yaml) |
| [musl-dynamic](https://github.com/chainguard-images/musl-dynamic) | [![CI status](https://github.com/chainguard-images/musl-dynamic/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/musl-dynamic/actions/workflows/release.yaml) |
| [nginx](https://github.com/chainguard-images/nginx) | [![CI status](https://github.com/chainguard-images/nginx/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/nginx/actions/workflows/release.yaml) |
| [php](https://github.com/chainguard-images/php) | [![CI status](https://github.com/chainguard-images/php/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/php/actions/workflows/release.yaml) |
| [python](https://github.com/chainguard-images/python) | [![CI status](https://github.com/chainguard-images/python/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/python/actions/workflows/release.yaml) |
| [sdk](https://github.com/chainguard-images/sdk) | [![CI status](https://github.com/chainguard-images/sdk/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/sdk/actions/workflows/release.yaml) |
| [static](https://github.com/chainguard-images/static) | [![CI status](https://github.com/chainguard-images/static/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/static/actions/workflows/release.yaml) |
| [wolfi-base](https://github.com/chainguard-images/wolfi-base) | [![CI status](https://github.com/chainguard-images/wolfi-base/actions/workflows/release.yaml/badge.svg)](https://github.com/chainguard-images/wolfi-base/actions/workflows/release.yaml) |
