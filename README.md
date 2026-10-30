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

The `app-teavm` and `component-teavm-worker` builds minify their JavaScript with
the shared Terser Maven execution in a release build (`-Prelease`) only, after
TeaVM compilation and before packaging. Every other build deploys TeaVM's
readable output with its source maps. TeaVM's own minification is disabled by
`instanto-teavm-pom`. Java source maps and asset filenames are preserved. A
release build needs Node.js and npm. Applications with custom TeaVM
executions should explicitly configure the corresponding `instanto.terser.*`
output properties and phase, and declare `exec-maven-plugin` after TeaVM and
before staging their assets.

See [PORTABILITY.md](PORTABILITY.md) for host integrations,
[FORMATTING.md](FORMATTING.md) for code style and
[SPOTBUGS.md](SPOTBUGS.md) for static analysis.

## Project links

[Sarto](https://github.com/instanto-io) ·
[TeaVM](https://github.com/konsoletyper/teavm)

To support maintenance: [Sponsor Sarto](https://github.com/sponsors/instanto-io)
or [Sponsor TeaVM](https://github.com/sponsors/konsoletyper).

## Quality

The shared build configuration is verified by the organisation's CI. Consumers publish a compact SpotBugs/SonarQube table on each verification run, and repositories with their own verification job also publish SARIF to code scanning.

See [Actions](https://github.com/instanto-io/sarto-poms/actions) for the latest verification result.

