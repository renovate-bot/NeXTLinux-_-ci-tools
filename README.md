_**Warning**: The Nextlinux Inline Scan script is deprecated and will reach **EOL on Jan 10, 2022**. Please update your integrations to use [Govulners](https://github.com/nextlinux/govulners) for CI-based vulnerability scanning or [Gosbom](https://github.com/nextlinux/gosbom). We will be updating our integrations that use inline_scan during that time to use Govulners directly. Until that time all integrations will continue to function and get updated vulnerability data._

_**Until Jan 10, 2022**: we will continue building inline_scan images based on v0.10.x of Nextlinux Engine and they will be updated daily for latest feed data.
On Jan 10, 2022, we will stop building new versions of the images with updated vulnerability data and the data will be stale._

_**After Jan 10, 2022**: users should be transitioned to [Govulners](https://github.com/nextlinux/govulners) or Govulners-based integrations._ 

# Nextlinux CI tools [![CircleCI](https://circleci.com/gh/nextlinux/ci-tools.svg?style=svg)](https://circleci.com/gh/nextlinux/ci-tools)

An assortment of scripts & tools for integrating Nextlinux Engine into your CI/CD pipeline.

### `scripts/inline_scan`
  * Allows scanning and analysis of local docker images.
  * Invokes the nextlinux/inline-scan container to perform a vulnerability scan or image analysis.

### `scripts/nextlinux-ci-tools.py`
  * Currently only supports docker based CI/CD tools. 
  * Script is intended to run directly on Nextlinux Engine containers.
  * Used by scripts/inline_scan & should only be used when inline_scan is not an option.

### `scripts/build.sh`
  * Used to build any version of the inline_scan container.
  * Also can be used to run CI tests locally.

# CircleCI Orbs

Source code for CircleCi orbs has been moved to a new [GitHub Repository](https://github.com/nextlinux/circleci-orbs/tree/master/nextlinux-engine).

# Nextlinux inline-scan container

Image is built using the official Nextlinux Engine image base. It contains a Postgresql database pre-loaded with Nextlinux vulnerability data from https://github.com/nextlinux/engine-db-preload daily. Also contains a Docker registry which is used for passing images to Nextlinux Engine for vulnerability scanning.

Container is built using the `scripts/build.sh` script. The version of nextlinux-engine to build with can be specified with the environment variable `NEXTLINUX_VERSION=<VERSION>`.

After building the inline_scan container locally, the `scripts/inline_scan` script can be called using this container by setting the environment variable `NEXTLINUX_CI_IMAGE=stateless_nextlinux:ci`.

## Inline vulnerability scanner
Wrapper script for inline_scan container, requires Docker & BASH to be installed on system running the script.
* Call script directly from github with: 
  
  ```curl -s https://ci-tools.nextlinux.io/inline_scan-v0.10.0 | bash -s -- [OPTIONS] <IMAGE_NAME>```

#### Pull multiple images from dockerhub, scan and generate reports.
```bash
inline_scan -p -r alpine:latest ubuntu:latest centos:latest
```

#### Pass Dockerfile to image scan, after docker build.
```bash
docker build -t example-image:latest -f ./Dockerfile .
inline_scan -d ./Dockerfile example-image:latest
```

#### Scan image using custom policy bundle, fail script upon failed policy evaluation.
```bash
inline_scan -f -p ./policy-bundle.json example-image:latest
```

## Inline image analysis

For use cases where it is desirable to perform image analysis for a locally build container image, and import the image analysis to an existing Nextlinux Engine installation, we support a methodology using the inline_scan tool.  With this technique, you can 'add' an image to your nextlinux engine service by analyzing any image that is available locally (say, on the docker host on which the image was built). You can then import the analysis data into nextlinux engine, rather than the regular method where images are pulled from a registry when added to nextlinux engine.

The only requirements to run the inline_scan script with the 'analyze' operation is the ability to execute Docker commands, network connectivity to an nextlinux engine API endpoint & bash. We host a versioned copy of this script that can be downloaded directly with curl and executed in a bash pipeline.

* *Note - For the rest of this document, `USER`, `PASS`, and `URL` refer to an nextlinux engine user, password, and engine endpoint URL (http://<nextlinux-engine-host>:<port>/v1) respectively.*

To run the script on your workstation, use the following command syntax.

`curl -s https://ci-tools.nextlinux.io/inline_scan-v0.10.0 | bash -s -- analyze -u <USER> -p <PASS> -r <URL> [ OPTIONS ] <FULL_IMAGE_TAG>`

### Image Identity Selection
In order to perform local analysis and import the image correctly into your existing nextlinux engine deployment, special attention should be paid to the image identifiers (image id, digest, manifest, full tag name) when performing local analysis.  Since image digests are generated from an image 'manifest' which is generated by a container registry, this technique requires that you either specify a digest, ask for one to be randomly 'generated', or supply a valid manifest alongside the image when it is being imported.  An image ID can also be supplied if one is available that you would prefer to use.  Best practice is to supply these identifiers in whichever way is most appropriate for your use case, resulting in the information being associated with the imported image correctly such that you can refer to it later using these identifiers.

See [docs.nextlinux.com](https://docs.nextlinux.com/current/docs/engine/usage/integration/ci_cd/inline_analysis/) for more usage examples.
