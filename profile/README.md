# Chainguard Images

Chainguard Images is a collection of container images designed for minimalism and security.

Many of these images are _distroless_; they contain only an application and its runtime
dependencies. There is no shell or package manager.

All these images are built using [apko](https://github.com/chainguard-dev/apko) and
[melange](https://github.com/chainguard-dev/melange). Combined, these tools provide for a
reproducible, declarative approach to building OCI images.

melange lets you build apks using declarative YAML pipelines. apks are .apk packages compatible with
the package manager used by Alpine, similar to .deb or .rpm for instance.

apko lets you bundle a collection of APKs into an OCI image using a declarative YAML manifest.

Chainguard Images provide SBOM support and signatures for known provenance and more secure base
images. They can be part of an approach to a secure software factory.

Many of the images use the [Wolfi undistro](https://github.com/wolfi-dev/), and we are working on
porting the rest.

## Find and use Chainguard Images

Our images are available via cgr.dev.

To pull the apko image with Docker:

```
docker pull cgr.dev/chainguard/apko 
```

All Chainguard Images are signed using [Sigstore](https://www.sigstore.dev/), and you can check
the signature using [`cosign`](https://docs.sigstore.dev/cosign/overview). For our apko image example,
you can run the following:

```
COSIGN_EXPERIMENTAL=1 cosign verify cgr.dev/chainguard/apko | jq 
```

Your output will indicate that the cosign claims were validated.

## Learn more 

You can learn more about Wolfi, distroless images, apko, and melange from the following
articles:

 - [Introducing apko: bringing distroless nirvana to Alpine Linux](https://blog.chainguard.dev/introducing-apko-bringing-distroless-nirvana-to-alpine-linux/)
 - [Minimal Container Images: Towards a More Secure Future](https://blog.chainguard.dev/minimal-container-images-towards-a-more-secure-future/)
 - [Secure Your Software Factory with melange and apko Media](https://blog.chainguard.dev/secure-your-software-factory-with-melange-and-apko/)

## Support

Support is provided on a best-effort basis via GitHub issues.

For commercial support, please see [Chainguard](https://www.chainguard.dev/chainguard-images).
