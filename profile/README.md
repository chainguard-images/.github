# Chainguard Images

<a href="https://github.com/chainguard-images/images#chainguard-images"><img width="133" src="https://raw.githubusercontent.com/chainguard-images/.github/main/profile/assets/view_images_button_4x.png"/></a>

** *CHAINGUARD OFFICE HOURS NOW AVAILABLE* ** <br>
[Book 1:1 office hours](https://get.chainguard.dev/chainguard-images-office-hours) to ask us anything about images or troubleshoot any issues. 
--
[Chainguard Images](https://www.chainguard.dev/chainguard-images) is a collection of container images designed for minimalism and security.

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

All of the images are based on the [Wolfi un-distro](https://github.com/wolfi-dev/), with the exception of:

- `cgr.dev/chainguard/static:latest`
- `cgr.dev/chainguard/busybox:latest`
- `cgr.dev/chainguard/git:latest`

...which are based on [Alpine Linux](https://www.alpinelinux.org/) and [musl](https://musl.libc.org/).

The above Alpine-based images have an equivalent Wolfi-based image, tagged `:latest-glibc`.

All Wolfi-based images are based on [glibc](https://www.gnu.org/software/libc/), with the exception of [`gcc-musl`](https://github.com/chainguard-images/images/tree/main/images/gcc-musl) and [`musl-dynamic`](https://github.com/chainguard-images/images/tree/main/images/musl-dynamic).

## Find and use Chainguard Images

Our images are available via cgr.dev.

For example, to pull the apko image with Docker:

```
docker pull cgr.dev/chainguard/apko 
```

There are a wide variety of images available. For more information, refer to the [authoritative list of images](https://github.com/chainguard-images/images) or [documentation](https://edu.chainguard.dev/chainguard/chainguard-images/). 

We'll cover some of the main categories and highlights below.

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
$ DOCKER_BUILDKIT=1 docker build -t c-static .
...
$ docker run c-static
Hello World!
```

In some cases it is easier to produce binaries that are dynamically linked against glibc or musl,
but otherwise self-contained. In those cases you can use the following images:

 - [glibc-dynamic](https://github.com/chainguard-images/images/tree/main/images/glibc-dynamic)
 - [musl-dynamic](https://github.com/chainguard-images/images/tree/main/images/musl-dynamic)

### Language Runtimes

These images can be used to run applications:

- [python](https://github.com/chainguard-images/images/tree/main/images/python)
- [node](https://github.com/chainguard-images/images/tree/main/images/node)
- [ruby](https://github.com/chainguard-images/images/tree/main/images/ruby)
- [java](https://github.com/chainguard-images/images/tree/main/images/jre)

### Compilers

These images can be used to compile artifacts for use in runtime or base images. For instance the go
image can be used to compile golang code for use in the static base image.

We have plans to include extra tooling in these images for creating provenance information such
as SBOMs.

Examples:

 - [go](https://github.com/chainguard-images/images/tree/main/images/go)
 - [gcc-glibc](https://github.com/chainguard-images/images/tree/main/images/gcc-glibc)
 - [gcc-musl](https://github.com/chainguard-images/images/tree/main/images/gcc-musl)
 - [jdk](https://github.com/chainguard-images/images/tree/main/images/jdk)
 - [maven](https://github.com/chainguard-images/images/tree/main/images/maven)
 - [bazel](https://github.com/chainguard-images/images/tree/main/images/bazel)

### Middleware and Applications

These images contain stand-alone software, such as databases and web servers, as well as tooling
such as apko and melange.

Because our images are constantly rebuilt with the latest sources and include the absolute minimum
of dependencies, they typically have significantly less vulnerabilities than equivalent images.

For example:
 - [nginx](https://github.com/chainguard-images/images/tree/main/images/nginx) 
 - [git](https://github.com/chainguard-images/images/tree/main/images/git)
 - [kubectl](https://github.com/chainguard-images/images/tree/main/images/kubectl)
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
$ cosign download attestation --predicate-type https://spdx.dev/Document --platform linux/amd64 cgr.dev/chainguard/nginx | jq -r .payload | base64 -d | jq
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

For more information on Chainguard images and how to use them visit [Chainguard Academy](https://edu.chainguard.dev/chainguard/chainguard-images/overview/). 

You can learn more about Wolfi, distroless images, apko, and melange from the following
articles:

 - [Introducing apko: bringing distroless nirvana to Alpine Linux](https://blog.chainguard.dev/introducing-apko-bringing-distroless-nirvana-to-alpine-linux/)
 - [Minimal Container Images: Towards a More Secure Future](https://blog.chainguard.dev/minimal-container-images-towards-a-more-secure-future/)
 - [Secure Your Software Factory with melange and apko](https://blog.chainguard.dev/secure-your-software-factory-with-melange-and-apko/)

## Support

Support is provided on a best-effort basis via GitHub issues.

For commercial support, please see [Chainguard](https://www.chainguard.dev/chainguard-images).
