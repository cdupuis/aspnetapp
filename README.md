# ASP.NET Core Docker Sample

This repository can be used to demo Docker Scout with an ASP.NET Core application.

## Build the image

Note: Enable containerd in your DD to make attestations work.

```shell
$ docker buildx b . -f Dockerfile.alpine \
  -t christiandupuis299/aspnetapp:main \
  --attest type=sbom,generator=docker/scout-sbom-indexer:1 \
  --provenance=1 --load --no-cache --platform linux/amd64,linux/arm64
```

## CVEs

This sample has been modified from original to include some .NET Core specific vulnerabilties:

```shell
$ docker scout cves christiandupuis299/aspnetapp:main --only-package-type nuget
    ✓ SBOM obtained from attestation, 335 packages found
    ✓ Provenance obtained from attestation
    ✗ Detected 2 vulnerable packages with a total of 4 vulnerabilities


## Overview

                    │                Analyzed Image
────────────────────┼───────────────────────────────────────────────
  Target            │  christiandupuis299/aspnetapp:main
    digest          │  ac768bd0366f
    platform        │ linux/arm64
    provenance      │ ssh://github.com/dotnet/dotnet-docker
                    │  7d4d56941607d8521d500be152d66bb7d9e3dbf0
    vulnerabilities │    0C     1H     3M     0L
    size            │ 55 MB
    packages        │ 313
                    │
  Base image        │  mcr.microsoft.com/dotnet/aspnet:8.0-alpine
                    │  25b1e6815f4f


## Packages and Vulnerabilities

   0C     1H     0M     0L  Npgsql 8.0.1.0
pkg:nuget/Npgsql@8.0.1.0

Dockerfile.alpine (22:22)
COPY --from=build /app .

    ✗ HIGH CVE-2024-32655 [Integer Overflow or Wraparound]
      https://scout.docker.com/v/CVE-2024-32655
      Affected range  : >=8.0.0
                      : <8.0.3
      Fixed version   : 8.0.3
      CVSS Score      : 8.1
      CVSS Vector     : CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H
      EPSS Score      : 0.05%
      EPSS Percentile : 17th percentile


   0C     0H     3M     0L  BouncyCastle.Cryptography 2.3.0.53016
pkg:nuget/BouncyCastle.Cryptography@2.3.0.53016

Dockerfile.alpine (22:22)
COPY --from=build /app .

    ✗ MEDIUM CVE-2024-30171 [Observable Discrepancy]
      https://scout.docker.com/v/CVE-2024-30171
      Affected range : <2.3.1
      Fixed version  : 2.3.1
      CVSS Score     : 5.9
      CVSS Vector    : CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:N/A:N

    ✗ MEDIUM CVE-2024-30172 [Loop with Unreachable Exit Condition ('Infinite Loop')]
      https://scout.docker.com/v/CVE-2024-30172
      Affected range : <2.3.1
      Fixed version  : 2.3.1
      CVSS Score     : 5.3
      CVSS Vector    : CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:L

    ✗ MEDIUM CVE-2024-29857 [Uncontrolled Resource Consumption]
      https://scout.docker.com/v/CVE-2024-29857
      Affected range : <2.3.1
      Fixed version  : 2.3.1
      CVSS Score     : 5.3
      CVSS Vector    : CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:L



4 vulnerabilities found in 2 packages
  LOW       0
  MEDIUM    3
  HIGH      1
  CRITICAL  0

```

## Policy

With containerd enabled, local policy evaluation will work and yield some interesting results:

```
docker scout policy christiandupuis299/aspnetapp:main
    ✓ SBOM obtained from attestation, 335 packages found
    ✓ Provenance obtained from attestation
    ✓ Policy evaluation completed


## Overview

             │           Analyzed Image
─────────────┼──────────────────────────────────────
  Target     │  christiandupuis299/aspnetapp:main
    digest   │  64f44fb1d7d4
    platform │ linux/arm64


## Policies

Policy status  FAILED  (6/9 policies met)

  Status │                     Policy                      │           Results
─────────┼─────────────────────────────────────────────────┼──────────────────────────────
  ✓      │ Copyleft licenses                               │    0 packages
  !      │ Default non-root user                           │
  !      │ Fixable critical and high CVEs (Production SLA) │    4 deviations
  ✓      │ Fixable critical and high vulnerabilities       │    0C     0H     0M     0L
  ✓      │ High-profile vulnerabilities                    │    0C     0H     0M     0L
  ✓      │ No embedded secrets                             │    0 deviations
  ✓      │ Outdated base images                            │
  ✓      │ Supply chain attestations                       │    0 deviations
  !      │ Unapproved base images                          │    1 deviation


## "Default non-root user" policy evaluation results
Ensures the image specifies a non-root username (or UID) for the final stage.

  User │ Explicit
───────┼───────────
  root │ false


## "Fixable critical and high CVEs (Production SLA)" policy evaluation results
This policy checks for fixable critical and high CVEs

                                Purl                               │ Vulnerability  │  Fixed by  │ Severity │ Score │     Epss
───────────────────────────────────────────────────────────────────┼────────────────┼────────────┼──────────┼───────┼───────────────
  pkg:apk/alpine/busybox@1.36.1-r15?os_name=alpine&os_version=3.19 │ CVE-2023-42366 │ 1.36.1-r16 │ MEDIUM   │ 5.50  │ 0.04% (0.12)
  pkg:apk/alpine/busybox@1.36.1-r15?os_name=alpine&os_version=3.19 │ CVE-2023-42363 │ 1.36.1-r17 │ MEDIUM   │ 5.50  │ 0.04% (0.12)
  pkg:apk/alpine/busybox@1.36.1-r15?os_name=alpine&os_version=3.19 │ CVE-2023-42364 │ 1.36.1-r17 │ MEDIUM   │ 5.50  │ 0.04% (0.12)
  pkg:apk/alpine/busybox@1.36.1-r15?os_name=alpine&os_version=3.19 │ CVE-2023-42365 │ 1.36.1-r17 │ MEDIUM   │ 5.50  │ 0.04% (0.12)


## "Unapproved base images" policy evaluation results
Base images must be from approved sources.

                  Base image                 │                   Reason
─────────────────────────────────────────────┼──────────────────────────────────────────────
  mcr.microsoft.com/dotnet/aspnet:8.0-alpine │ Does not match any approved glob expression

```

--- 

This sample demonstrates how to build container images for ASP.NET Core web apps. See [.NET Docker Samples](../README.md) for more samples.

> Note: .NET 8 container images use port `8080`, by default. Previous .NET versions used port `80`. The instructions for the sample assume the use of port `8080`.

## Run the sample image

You can start by launching a sample from our [container registry](https://mcr.microsoft.com/) and access it in your web browser at `http://localhost:8000`.

```console
docker run --rm -it -p 8000:8080 -e ASPNETCORE_HTTP_PORTS=8080 mcr.microsoft.com/dotnet/samples:aspnetapp
```

You can also call an endpoint that the app exposes:

```bash
$ curl http://localhost:8000/Environment
{"runtimeVersion":".NET 8.0.0-preview.6.23329.7","osVersion":"Ubuntu 22.04.2 LTS","osArchitecture":"Arm64","user":"app","processorCount":4,"totalAvailableMemoryBytes":4124442624,"memoryLimit":0,"memoryUsage":31518720,"hostName":"78e2b2cfc0e8"}
```

This container image is built with [Ubuntu Chiseled](https://devblogs.microsoft.com/dotnet/dotnet-6-is-now-in-ubuntu-2204/#net-in-chiseled-ubuntu-containers), with [Dockerfile](Dockerfile.chiseled-composite).

## Change port

You can change the port ASP.NET Core uses with one of the following environment variables. However, port `8080` (set by default) is recommended.

The following examples change the port to port `80`.

Supported with .NET 8+:

```bash
ASPNETCORE_HTTP_PORTS=80
```

Supported with .NET Core 1.0+

```bash
ASPNETCORE_URLS=http://+:80 
```

Note: `ASPNETCORE_URLS` overwrites `ASPNETCORE_HTTP_PORTS` if set.

These environment variables are used in [.NET 8](https://github.com/dotnet/dotnet-docker/blob/6da64f31944bb16ecde5495b6a53fc170fbe100d/src/runtime-deps/8.0/bookworm-slim/amd64/Dockerfile#L7C5-L7C31) and [.NET 6](https://github.com/dotnet/dotnet-docker/blob/6da64f31944bb16ecde5495b6a53fc170fbe100d/src/runtime-deps/6.0/bookworm-slim/amd64/Dockerfile#L5) Dockerfiles, respectively.

## Build image

You can built an image using one of the provided Dockerfiles.

```console
docker build --pull -t aspnetapp .
docker run --rm -it -p 8000:8080 -e ASPNETCORE_HTTP_PORTS=8080 aspnetapp
```

You should see the following console output as the application starts:

```console
> docker run --rm -it -p 8000:8080 -e ASPNETCORE_HTTP_PORTS=8080 aspnetapp
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://[::]:8080
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
```

After the application starts, navigate to `http://localhost:8000` in your web browser. You can also view the ASP.NET Core site running in the container from another machine with a local IP address such as `http://192.168.1.18:8000`.

> Note: ASP.NET Core apps (in official images) listen to [port 8080 by default](https://github.com/dotnet/dotnet-docker/blob/6da64f31944bb16ecde5495b6a53fc170fbe100d/src/runtime-deps/8.0/bookworm-slim/amd64/Dockerfile#L7), starting with .NET 8. The [`-p` argument](https://docs.docker.com/engine/reference/commandline/run/#publish) in these examples maps host port `8000` to container port `8080` (`host:container` mapping). The container will not be accessible without this mapping. ASP.NET Core can be [configured to listen on a different or additional port](https://learn.microsoft.com/aspnet/core/fundamentals/servers/kestrel/endpoints).

You can see the app running via `docker ps`.

```bash
$ docker ps
CONTAINER ID   IMAGE                                        COMMAND         CREATED          STATUS                    PORTS                  NAMES
d79edc6bfcb6   mcr.microsoft.com/dotnet/samples:aspnetapp   "./aspnetapp"   35 seconds ago   Up 34 seconds (healthy)   0.0.0.0:8080->8080/tcp   nice_curran
```

You may notice that the sample includes a [health check](../enable-healthchecks.md), indicated in the "STATUS" column.

## Build image with the SDK

The easiest way to [build images is with the SDK](https://github.com/dotnet/sdk-container-builds). 

```console
dotnet publish /p:PublishProfile=DefaultContainer
```

That command can be further customized to use a different base image and publish to a container registry. You must first use `docker login` to login to the registry.

```console
dotnet publish /p:PublishProfile=DefaultContainer /p:ContainerBaseImage=mcr.microsoft.com/dotnet/aspnet:8.0-jammy-chiseled /p:ContainerRegistry=docker.io /p:ContainerRepository=youraccount/aspnetapp
```

## Supported Linux distros

The .NET Team publishes images for [multiple distros](../../documentation/supported-platforms.md).

Samples are provided for:

- [Alpine](Dockerfile.alpine)
- [Alpine with Composite ready-to-run image](Dockerfile.alpine-composite)
- [Alpine with ICU installed](Dockerfile.alpine-icu)
- [Debian](Dockerfile.debian)
- [Ubuntu](Dockerfile.ubuntu)
- [Ubuntu Chiseled](Dockerfile.chiseled)
- [Ubuntu Chiseled with Composite ready-to-run image](Dockerfile.chiseled-composite)

## Supported Windows versions

The .NET Team publishes images for [multiple Windows versions](../../documentation/supported-platforms.md). You must have [Windows containers enabled](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers) to use these images.

Samples are provided for

- [Nano Server](Dockerfile.nanoserver)
- [Windows Server Core](Dockerfile.windowsservercore)
- [Windows Server Core with IIS](Dockerfile.windowsservercore-iis)

Windows variants of the sample can be pulled via one the following registry addresses:

- `mcr.microsoft.com/dotnet/samples:aspnetapp-nanoserver-1809`
- `mcr.microsoft.com/dotnet/samples:aspnetapp-nanoserver-ltsc2022`
