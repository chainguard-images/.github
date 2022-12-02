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

 - [glibc-dynamic](https://github.com/chainguard-images/images/tree/main/images/glibc-dynamic)
 - [musl-dynamic](https://github.com/chainguard-images/images/tree/main/images/musl-dynamic)

### Language Runtimes

These are still largely in progress, but we are working on images for runtimes such as the JRE.

### Compilers

These images can be used to compile artifacts for use in runtime or base images. For instance the go
image can be used to compile golang code for use in the static base image.

We have plans to include extra tooling in these images for creating provenance information such
as SBOMs.

Examples:

 - [go](https://github.com/chainguard-images/images/tree/main/images/go)
 - [gcc-glibc](https://github.com/chainguard-images/images/tree/main/images/gcc-glibc)
 - [gcc-musl](https://github.com/chainguard-images/images/tree/main/images/gcc-musl)

### Middleware and Applications

These images contain stand-alone software, such as databases and web servers, as well as tooling
such as apko and melange.

Because our images are constantly rebuilt with the latest sources and include the absolute minimum
of dependencies, they typically have significantly less vulnerabilities than equivalent images.

For example:
 - [nginx](https://github.com/chainguard-images/images/tree/main/images/nginx) 
 - [git](https://github.com/chainguard-images/images/tree/main/images/git)
 - [apko](https://github.com/chainguard-images/images/tree/main/images/apko)
 - [melange](https://github.com/chainguard-images/images/tree/main/images/melange)

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

| Name | OCI Reference | Variants/Tags |
| ----- | ----- |  -------- |
| [alpine-base](./images/alpine-base) | `cgr.dev/chainguard/alpine-base` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/alpine-base.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/alpine-base:latest) |
| [apko](./images/apko) | `cgr.dev/chainguard/apko` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/apko.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/apko:latest) |
| [bazel](./images/bazel) | `cgr.dev/chainguard/bazel` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/bazel.build.status.experimental.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/bazel:experimental) |
| [busybox](./images/busybox) | `cgr.dev/chainguard/busybox` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/busybox.build.status.latest-glibc.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/busybox:latest-glibc)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/busybox.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/busybox:latest) |
| [gcc-glibc](./images/gcc-glibc) | `cgr.dev/chainguard/gcc-glibc` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/gcc-glibc.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/gcc-glibc:latest) |
| [gcc-musl](./images/gcc-musl) | `cgr.dev/chainguard/gcc-musl` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/gcc-musl.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/gcc-musl:latest) |
| [git](./images/git) | `cgr.dev/chainguard/git` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/git.build.status.latest-glibc-root.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/git:latest-glibc-root)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/git.build.status.latest-glibc.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/git:latest-glibc)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/git.build.status.latest-root.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/git:latest-root)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/git.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/git:latest) |
| [glibc-dynamic](./images/glibc-dynamic) | `cgr.dev/chainguard/glibc-dynamic` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/glibc-dynamic.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/glibc-dynamic:latest) |
| [go](./images/go) | `cgr.dev/chainguard/go` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/go.build.status.latest-glibc.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/go:latest-glibc)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/go.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/go:latest) |
| [jdk](./images/jdk) | `cgr.dev/chainguard/jdk` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/jdk.build.status.openjdk-11.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/jdk:openjdk-11)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/jdk.build.status.openjdk-17.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/jdk:openjdk-17) |
| [jenkins](./images/jenkins) | `cgr.dev/chainguard/jenkins` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/jenkins.build.status.experimental.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/jenkins:experimental) |
| [jre](./images/jre) | `cgr.dev/chainguard/jre` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/jre.build.status.openjdk-jre-11.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/jre:openjdk-jre-11)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/jre.build.status.openjdk-jre-17.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/jre:openjdk-jre-17) |
| [ko](./images/ko) | `cgr.dev/chainguard/ko` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/ko.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/ko:latest) |
| [kubectl](./images/kubectl) | `cgr.dev/chainguard/kubectl` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/kubectl.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/kubectl:latest) |
| [maven](./images/maven) | `cgr.dev/chainguard/maven` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/maven.build.status.openjdk-11.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/maven:openjdk-11)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/maven.build.status.openjdk-17.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/maven:openjdk-17) |
| [melange](./images/melange) | `cgr.dev/chainguard/melange` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/melange.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/melange:latest) |
| [musl-dynamic](./images/musl-dynamic) | `cgr.dev/chainguard/musl-dynamic` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/musl-dynamic.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/musl-dynamic:latest) |
| [nginx](./images/nginx) | `cgr.dev/chainguard/nginx` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/nginx.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/nginx:latest)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/nginx.build.status.stable.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/nginx:stable) |
| [node](./images/node) | `cgr.dev/chainguard/node` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/node.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/node:latest) |
| [php](./images/php) | `cgr.dev/chainguard/php` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/php.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/php:latest) |
| [postgres](./images/postgres) | `cgr.dev/chainguard/postgres` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/postgres.build.status.15.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/postgres:15) |
| [python](./images/python) | `cgr.dev/chainguard/python` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/python.build.status.latest-glibc.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/python:latest-glibc)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/python.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/python:latest) |
| [ruby](./images/ruby) | `cgr.dev/chainguard/ruby` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/ruby.build.status.latest-3.0.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/ruby:latest-3.0)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/ruby.build.status.latest-3.1.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/ruby:latest-3.1) |
| [sdk](./images/sdk) | `cgr.dev/chainguard/sdk` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/sdk.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/sdk:latest)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/sdk.build.status.wolfi.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/sdk:wolfi) |
| [static](./images/static) | `cgr.dev/chainguard/static` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/static.build.status.latest-glibc.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/static:latest-glibc)<br/>[![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/static.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/static:latest) |
| [wolfi-base](./images/wolfi-base) | `cgr.dev/chainguard/wolfi-base` | [![](https://storage.googleapis.com/chainguard-images-build-outputs/badges/wolfi-base.build.status.latest.svg)](https://registry-ui.chainguard.app/?image=cgr.dev/chainguard/wolfi-base:latest) |
