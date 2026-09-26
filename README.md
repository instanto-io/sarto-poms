# sarto-poms — Reusable Parent POMs

Parent and version-management POMs for the Sarto libraries and the Sarto
application development frameworks. This repository publishes no runtime JARs.

Each POM documents its own role, what it provides to downstream projects and
what it requires from them. Read the POM for detail; this file is only a map.

## Hierarchy

```text
instanto-org-pom                   Common Instanto build and release settings
└── instanto-teavm-pom             TeaVM version, dependency set and plugin pin
    ├── sarto-library-pom          A Sarto library released on its own
    └── sarto-org-pom              Sarto dependency versions, licence, publication
        ├── sarto-framework-pom    The reusable framework modules
        ├── sarto-entity-model-pom Managed entity model modules
        │   └── sarto-service-pom  Server-side service modules
        └── sarto-app-pom          Portable applications, JRE and TeaVM
            ├── sarto-app-browser-pom     Served as static files
            ├── sarto-app-cloudflare-pom  Cloudflare Workers and Pages
            └── sarto-app-tomcat-pom      Servlet container WAR
```

`instanto-org-pom` and `instanto-teavm-pom` come from the standalone
[`instanto-poms`](https://github.com/instanto-io/instanto-poms) repository.

## Which parent

| Building | Parent |
|---|---|
| A library released on its own, with no framework version set | `sarto-library-pom` |
| A reusable framework module | `sarto-framework-pom` |
| A managed entity model shared by client and server | `sarto-entity-model-pom` |
| A server-side service | `sarto-service-pom` |
| A portable application | one of the `sarto-app-*-pom` hosting parents |

## Classifier policy

| Classifier | Meaning |
|---|---|
| *(none)* | Runtime-neutral — safe on both the JVM and TeaVM classpath |
| `jre` | JVM only |
| `teavm` | TeaVM only (JSO or substitution based) |
| `web` | Packaged TeaVM web bundle (zip) |
| `shared` | Small cross-runtime metadata artifact |

## Further guidance

See [PORTABILITY.md](PORTABILITY.md) for host integrations,
[FORMATTING.md](FORMATTING.md) for code style and
[SPOTBUGS.md](SPOTBUGS.md) for static analysis.

## Project links

[Sarto](https://github.com/cstainton) ·
[TeaVM](https://github.com/konsoletyper/teavm)

To support maintenance: [Sponsor Sarto](https://github.com/sponsors/cstainton)
or [Sponsor TeaVM](https://github.com/sponsors/konsoletyper).
