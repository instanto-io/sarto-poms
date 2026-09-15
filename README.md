# sarto-poms — Reusable Parent POMs

Reusable POMs for the Sarto libraries and Sarto application development
frameworks, published from the standalone `sarto-poms` repository. All
artifacts share the suite's `${revision}` version, defined in the root `pom.xml`.

---

## JVM and TeaVM

This repository publishes parent and version-management POMs, not runtime
JARs. Application parents configure a JVM, browser, or Worker build.
The framework parent adds the portable, JRE, and TeaVM source roots to a
module's build when those directories are present.

A reusable capability publishes one primary JAR containing its shared code
and any indexed platform implementations. The application build prepares its
target's entry point. Build plugins and TeaVM compiler support remain build
dependencies. Runtime JARs retain their ordinary Maven dependencies.

See [independent library portability](PORTABILITY.md) for the repository
host integrations, application setup and test commands.

## Hierarchy

The shared organisation parent is `io.instanto:instanto-org-pom`, maintained
in the standalone `instanto-poms` project. Install it first with
`mvn -f ../instanto-poms/pom.xml install` when working with local snapshots.
Both parent chains below inherit its common build and release settings:

```
instanto-org-pom                                      — common organisation build and release settings
├── sarto-library-pom  (sarto-library-pom/pom.xml)       — independent library verification
└── sarto-org-pom      (sarto-org-pom/pom.xml)           — Sarto dependency and framework settings
    └── sarto-java-pom (sarto-java-pom/pom.xml)      — compiler, TeaVM and logging versions
        ├── sarto-framework-pom (sarto-framework-pom/pom.xml) — reusable Sarto framework modules
        ├── sarto-entity-model-pom (sarto-entity-model-pom/pom.xml)
        │     └── sarto-service-pom (sarto-service-pom/pom.xml)
        └── sarto-app-pom (sarto-app-pom/pom.xml)     — portable application build
              ├── sarto-app-cloudflare-pom (sarto-app-cloudflare-pom/pom.xml)
              ├── sarto-app-browser-pom (sarto-app-browser-pom/pom.xml)
              └── sarto-app-tomcat-pom (sarto-app-tomcat-pom/pom.xml)
```

Verrai application parents are released from the standalone
[`verrai`](https://github.com/cstainton/verrai) repository. They consume this
Sarto parent chain as normal Maven artifacts rather than through relative
paths.

---

## Parent Reference

### `sarto-library-pom` — independent library builds

Auth, REST, Config and Graphics inherit this parent through their repository aggregators;
the monorepo's build plugin also uses it directly.
It inherits build settings from `instanto-org-pom` and supplies library
verification without runtime dependencies or BOM imports. The organisation
parent and the Sarto POM suite have independent versions.

Versioned releases run from the Instanto-io checkout on local infrastructure.
The shared `release` profile supplies the release checks and source/Javadoc
packaging. Refined source is transferred from private development to Instanto-io
before a release begins. See the common parent's
`RELEASING.md` for branch creation, publication and the next snapshot version.

Unified library modules opt in by declaring the following plugins. The parent owns their
versions and lifecycle bindings; the module supplies its verification settings.

```xml
<plugin>
  <groupId>io.instanto</groupId>
  <artifactId>sarto-build-maven-plugin</artifactId>
</plugin>
```

For the source and Javadoc plugins, use group `org.apache.maven.plugins` and artifact IDs
`maven-source-plugin` and `maven-javadoc-plugin`. For the verifier, use
`io.instanto:sarto-build-maven-plugin`. Declaring all three runs the common
`verify-library` goal during `mvn verify`; aggregators and modules that do not opt in
are not checked as unified JARs. See the verifier README in the Sarto monorepo for
the checks and its two library-specific options.

For local snapshot builds, first install this POM with
`mvn -N -f sarto-library-pom/pom.xml install` and install `sarto-build-maven-plugin`
from the monorepo. Published builds resolve both as ordinary build dependencies.

### `sarto-framework-pom` — Sarto framework modules

Use for reusable modules that implement the Sarto framework and have no runtime
entry point. Independent libraries use the build-only `sarto-library-pom` instead.

```xml
<parent>
  <groupId>io.instanto</groupId>
  <artifactId>sarto-framework-pom</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <relativePath/>
</parent>
```

Provides: Java 21 compiler settings, `${teavm.version}`, `${slf4j.version}`.

---

### `sarto-entity-model-pom` — reusable managed entity model modules

Use for modules that publish managed `EntityModel` state without service RPC
generation.

```xml
<parent>
  <groupId>io.instanto</groupId>
  <artifactId>sarto-entity-model-pom</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <relativePath/>
</parent>
```

Provides annotation processor wiring for observable entities, automap, generated
entity providers, configuration entity schemas, and generated manager
registration. Source code still declares the model/entity boundaries; the POM
only enables the generation lane.

---

### `sarto-service-pom` — server-side service modules

Use for Sarto service contract and implementation modules. The standard service
triad gives those artifacts different dependency boundaries:

- `<name>-client` contains the Sarto contract, portable messages, and generated
  Sarto bindings. It stays host-neutral.
- `<name>` contains the deployable implementation and may use runtime facilities
  needed to do its job. A service that wraps an external HTTP API normally owns
  its generated Sarto REST client and the adapter for its declared runtime here.
- `<name>-all` is only the convenience aggregate for co-location.

Add `sarto-rest-client` to a service implementation that calls an HTTP API.
The same client JAR contains the JVM and TeaVM transports. The application
build selects its transport when preparing the target's CDI graph. Keep
generated REST client code in the implementation module; the service's
`*-client` artifact continues to expose its Sarto contract.

```xml
<parent>
  <groupId>io.instanto</groupId>
  <artifactId>sarto-service-pom</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <relativePath/>
</parent>
```

---

### `sarto-app-pom` — portable multi-runtime applications

Use for modules that build for more than one runtime target via Maven profiles.
Provides plugin management for TeaVM, JavaFX, and Swing assembly.

```xml
<parent>
  <groupId>io.instanto</groupId>
  <artifactId>sarto-app-pom</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <relativePath/>
</parent>
```

Profiles (all active by default unless noted):

| Profile | What it does |
|---|---|
| `app-jre` | Adds `src/jre/java` and the JVM CDI adapter; attaches `-jre` classified jar |
| `app-teavm` | Adds `src/teavm/java` and the TeaVM CDI adapter; runs `teavm-maven-plugin` |
| `app-jre-swing` | Swing assembly plugin defaults (`jar-with-dependencies`) |
| `app-jre-fx` | JavaFX dep + plugin defaults; fat-jar named `…-<platform>-all.jar` |
| `javafx-mac-arm` / `javafx-mac-x64` / `javafx-linux-x64` … | Platform classifier for JavaFX natives |

Required properties in child module:

```xml
<properties>
  <app.teavm.main.class>com.example.MyTeaVmApp</app.teavm.main.class>
  <app.teavm.target.file>app.js</app.teavm.target.file>
</properties>
```

---

### `sarto-app-cloudflare-pom` — Cloudflare-hosted applications

Select this parent in a deployment module when the same portable application is
to be compiled with TeaVM and staged as a Cloudflare Worker:

```xml
<parent>
  <groupId>io.instanto</groupId>
  <artifactId>sarto-app-cloudflare-pom</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <relativePath/>
</parent>
```

Application Java declares its Worker name, bindings, assets and non-secret
variables on its `WorkerEnvironment`. The Cloudflare generator turns that into
Wrangler configuration and validates missing resource identities during the
build. Secrets remain deployment inputs. `sarto-edge-cf-cdi` supplies the
TeaVM CDI runtime transitively; REST remains an optional deployment capability.
This parent is the first host-specific
member of the intended `app-*-host` family; JVM host parents should provide the
same boundary for Spring and Quarkus rather than adding host configuration to
the portable application modules.

---

### `sarto-app-browser-pom` — TeaVM browser applications and reusable WebJars

Use for browser applications that compile Java to JavaScript. The default
output is packaged below `META-INF/resources` in a `web`-classified jar, so a
Spring, Quarkus or Servlet deployment can co-locate and serve it simply by
depending on that artifact. The browser remains independently testable and
deployable as static assets.

```xml
<parent>
  <groupId>io.instanto</groupId>
  <artifactId>sarto-app-browser-pom</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <relativePath/>
</parent>
```

Additional defaults over `sarto-app`:

| Property | Default |
|---|---|
| `app.teavm.target.directory` | `target/classes/META-INF/resources` |
| `app.teavm.bundle.directory` | same as above |
| `app.teavm.phase` | `prepare-package` |

The attached `web` jar contains only `META-INF/resources/**`. WAR packaging is
still supported, with `failOnMissingWebXml=false`.

---

### `sarto-app-tomcat-pom` — Servlet/Tomcat WAR deployments

Use in a terminal deployment module and set `<packaging>war</packaging>`. The
parent supplies the Sarto Tomcat runtime and Servlet WAR conventions. A browser
module built with `sarto-app-browser-pom` may be added as a normal dependency; its
`META-INF/resources` are served by Tomcat from `WEB-INF/lib`.

The Tomcat runtime selects Sarto CDI for the JVM. Outbound REST remains an
optional deployment capability, so neither the parent nor application feature
modules add it unconditionally.

```xml
<parent>
  <groupId>io.instanto</groupId>
  <artifactId>sarto-app-tomcat-pom</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <relativePath/>
</parent>
<packaging>war</packaging>
```

Relay hosting is intentionally not another parent hierarchy. A relay is an
application role, while Spring, Quarkus, Tomcat and Cloudflare are execution
hosts. A dedicated relay distribution selects one `app-*-host` parent and adds
the appropriate relay-host dependency. An ordinary application can add the
same role when co-location is desired, without creating combinations such as
`app-spring-relay-host` and `app-tomcat-relay-host`.

---

### Standalone Verrai application parents

Use `sarto-verrai-app` for browser applications built with the Verrai
page/navigation framework. The parent is published by the standalone Verrai
repository and extends the Sarto application parent through Maven coordinates.

```xml
<parent>
  <groupId>io.instanto</groupId>
  <artifactId>sarto-verrai-app</artifactId>
  <version>0.1.0-SNAPSHOT</version>
  <relativePath/>
</parent>
<packaging>war</packaging>
<properties>
  <app.teavm.main.class>com.example.client.App</app.teavm.main.class>
  <app.teavm.target.file>app.js</app.teavm.target.file>
</properties>
```

What this parent adds over `sarto-app-browser-pom`:

**Annotation processors wired automatically** — no `<annotationProcessorPaths>`
needed in the child module:
- `sarto-codegen` — generates `_Stub`, `_Dispatcher`, `GeneratedAppManifest`
- `verrai-annotation-processor` — generates `_Factory`, `NavigationImpl_Factory`, `BootstrapperImpl`

**Server-facing route artifact** — the Verrai navigation processor generates
`io.instanto.verrai.impl.GeneratedRouteRegistry`, and this parent packages it in
a `shared` classifier when present. Server modules can depend on the client
module's `shared` classifier to resolve clean paths without bringing in the full
TeaVM client.

**Managed dependency versions** (add to `<dependencies>` without a `<version>`):

| artifactId | Purpose |
|---|---|
| `verrai-demo` | Page lifecycle, IoC container bootstrap |
| `verrai-sarto` | Injectable `ClientConnection`, login/session registration |
| `sarto-core` | `@Service`, `@Portable`, envelope model |
| `sarto-rpc` | `RpcInvocationHandler` |
| `sarto-codec-json` | JSON envelope serializer |
| `sarto-transport-stomp` | Unified STOMP contracts and target-selected JVM/TeaVM adapters |
| `sarto-codegen` | APT processor (processor path) |
| `verrai-annotation-processor` | APT processor (processor path) |

---

### `sarto-verrai-feature` and `sarto-verrai-shell` — multi-module Verrai clients

Use these only when a Verrai client is split across multiple Maven modules.

`sarto-verrai-feature` is for page-bearing feature modules. It compiles
`@Page` classes and emits page index resources, but does not generate the final
`NavigationImpl`, `GeneratedVerraiMain`, or shared route artifact.

`sarto-verrai-shell` is for the final shell module. It scans sibling feature
module outputs under the client aggregator for
`target/classes/META-INF/verrai/pages`, aggregates those pages with its own, and
generates the final browser runtime plus the server-consumable shared route jar.

Feature modules must be on the shell module's compile classpath. After moving or
deleting feature modules, prefer a clean build so old page indexes do not remain
in stale `target` directories.

---

## Classifier Policy

| Classifier | Meaning |
|---|---|
| *(none)* | Runtime-neutral artifact — safe for both JVM and TeaVM classpath |
| `jre` | JVM-only artifact |
| `teavm` | TeaVM-only artifact (JSO/substitution-based) |
| `web` | Packaged TeaVM web bundle (zip) |
| `shared` | Small cross-runtime metadata artifact, such as Verrai's generated route registry |

---

## Build Commands

Install the full POM chain (required before building any module in isolation):

```bash
./mvnw install -N -f sarto-org-pom/pom.xml
./mvnw install -N -f sarto-java-pom/pom.xml
./mvnw install -N -f sarto-framework-pom/pom.xml
./mvnw install -N -f sarto-entity-model-pom/pom.xml
./mvnw install -N -f sarto-service-pom/pom.xml
./mvnw install -N -f sarto-app-pom/pom.xml
./mvnw install -N -f sarto-app-cloudflare-pom/pom.xml
./mvnw install -N -f sarto-app-browser-pom/pom.xml
./mvnw install -N -f sarto-app-tomcat-pom/pom.xml
```

Or install the whole chain at once via the aggregator:

```bash
./mvnw install -DskipTests
```

For a Verrai application, also install the sibling standalone reactor when
using local snapshots:

```bash
mvn install -f ../verrai/pom.xml -DskipTests
```

### Verify dependency signatures before TeaVM

The org parent provides an input-signature verification profile:

```bash
mvn verify -P verify-input-signatures
```

It runs during Maven's `validate` phase, before javac, annotation processing,
Sarto code generation, and TeaVM. The first check verifies detached OpenPGP
signatures for resolved releases, including provided dependencies, annotation
processors, build plugins, and plugin dependencies. Current-reactor artifacts
and snapshots are excluded because they do not yet have release signatures.
The second check reads resolved JAR contents through the JVM signed-JAR verifier,
rejects broken or partially signed JARs, enforces any configured signer
fingerprints, and writes `target/sarto/input-artifacts.json` with coordinates
and SHA-256 hashes.

Configure the key policy for the build inputs being verified. The OpenPGP and
embedded-JAR checks report separate results.

For local snapshot builds, install `sarto-build-maven-plugin` from the Sarto
monorepo first. Maven resolves published plugin versions from the configured
package repository.

---

## Project links

[Sarto](https://github.com/cstainton) ·
[TeaVM](https://github.com/konsoletyper/teavm)

To support maintenance: [Sponsor Sarto](https://github.com/sponsors/cstainton)
or [Sponsor TeaVM](https://github.com/sponsors/konsoletyper).
