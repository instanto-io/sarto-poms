# Independent Sarto libraries on the JVM and TeaVM

This guide covers the 24 independent `sarto-*` repositories. It does not cover
the `sarto` monorepo. Each repository's README describes its own runtime and
build requirements.

## Add a library

A shared capability has one primary JAR for JVM and TeaVM consumers. It can
contain shared classes and classes for each platform. Add its ordinary Maven
dependency in either application. Maven also supplies the dependencies
declared by that library; these are not shaded, self-contained executable JARs.

A library does not start an application. The application owns startup and
chooses its platform implementation. For direct use, call the documented
constructor or factory for the host. CDI is optional unless the particular
integration module requires it.

For Sarto CDI, prepare a graph for the application's target. The
[bootstrap guide](https://github.com/cstainton/sarto-cdi/blob/main/sarto-cdi-bootstrap/README.md#select-platform-implementations)
shows the configuration. Application mode alone does not select platform
beans. A JVM graph includes JVM and shared beans; a TeaVM graph includes
TeaVM and shared beans. Declare both graphs only when building both targets.
Reusable libraries contribute their bean indexes, not application startup
providers.

TeaVM compiler integration remains part of the application build. Examples
include `sarto-cdi-teavm-tooling` for CDI and `teavm-extras-slf4j` for Model's
logging. These are separate from choosing a platform coordinate for each
runtime library.

## Choose a host integration

Some capabilities need a particular host. A JavaFX renderer needs JavaFX;
a Worker binding needs a Worker; an IndexedDB provider needs a browser.
Their contracts and clients can still be shared. An HTTP client calling a
Worker endpoint does not need the Worker implementation JAR.

Choose the library and host integration that match the application:

| Repository | Libraries and host requirements |
| --- | --- |
| `sarto-async` | Shared results, streams and stages. JVM futures and JavaScript promises have their own adapters; TeaVM compatibility support belongs to the build. |
| `sarto-auth` | Shared authentication contracts, client transports and testkit. Hosting and cryptographic integrations retain their JVM or Worker requirements. |
| `sarto-cdi` | Shared runtime and bootstrap JAR, including `JvmCdi` and `TeaVmCdi`. Processors, Maven integration and TeaVM compiler tooling run during the build. |
| `sarto-codec` | Shared codec runtime and generated codecs. Code generators run during Java compilation. |
| `sarto-config` | Shared configuration, parsers, typed access, CDI facade and testkit. The neutral `sarto-config` JAR includes a portable annotated settings service, JVM reflective mapping and TeaVM-generated mapping. Applications supply a resource source; the no-argument constructor reads the JVM classpath. |
| `sarto-edge` | Worker implementation modules require TeaVM and Worker bindings. Browser APIs require a browser. Build plugins and test runners run on the JVM. Use portable protocol clients to call deployed endpoints. |
| `sarto-events` | Shared events and one testkit containing both platforms' mock producers. |
| `sarto-geo` | Shared coordinates, projections and GeoJSON model integration. The indexed JVM GeoJSON reader adds file, URL and classpath access. |
| `sarto-geometry` | Shared geometry values and operations. JTS is a separate geometry capability, also tested with TeaVM. |
| `sarto-graphics` | Shared drawing, input, viewport and testkit contracts. AWT, JavaFX and browser canvas remain separate host adapters. |
| `sarto-grpc` | Shared client contracts and JVM/Fetch client transports. JVM and Worker server hosting remain distinct. Generators run during the build. |
| `sarto-model` | Shared model, XML, YAML and testkit libraries. TeaVM uses registered factories and supplied content; reflective construction and JVM content loading remain JVM conveniences. |
| `sarto-monitor-ui` | Browser UI and theme modules. These require browser APIs, not a JVM UI host. |
| `sarto-persistence` | Neutral runtime JAR containing JVM Jakarta and TeaVM promise/key-value support over portable API/mapping dependencies. TeaVM discovery excludes JVM-target providers before reachability. IndexedDB, memory and Worker storage remain separate artifacts; selecting and configuring a database remains a host decision. |
| `sarto-poms` | Parent and version-management POMs; no runtime JAR. |
| `sarto-relay` | Shared HTTP and STOMP clients. Worker relay hosting and JVM Artemis hosting remain host-specific. |
| `sarto-resources` | Shared resource pipeline, contracts, CDI facade and testkit. JVM resource acquisition remains a separate host integration. |
| `sarto-rest` | Shared REST clients, transport implementations and testkit. Servlet hosting and build generators keep their own requirements. |
| `sarto-scene` | Shared scene contracts and implementation. JavaFX and browser hosts remain separate. |
| `sarto-signals` | Shared signal and observation libraries, without a host startup requirement. |
| `sarto-site` | Documentation website; no Java runtime artifact. |
| `sarto-soap` | Shared SOAP client and protocol libraries. Schema and client generators run during the build. |
| `sarto-transactions` | Shared transaction runtime and testkit. The CDI extension runs during compilation; resource providers supply transaction behavior. |
| `sarto-xml` | Shared XML artifact containing JVM and TeaVM parsers. KXML remains a visible Maven dependency. |

## Test a shared library

Run JVM and TeaVM tests against the same production artifact. JVM tests live
under `src/test/jre/java` and TeaVM tests under `src/test/teavm/java` in the
testkits. Shared test code goes under `src/test/java`.

The Auth, REST and Transactions testkits publish their portable doubles
and both mock implementations in one JAR. Add the testkit with test scope.
Mockito and Mockatcha are declared dependencies; a generated TeaVM application
must not reach Mockito's classes. Direct tests can instantiate the appropriate
mock producer without CDI.

Auth and REST use the shared `sarto-build-maven-plugin:verify-library` goal
during `mvn verify`, with bindings inherited from the build-only `sarto-library-pom`.
The goal checks packaged dependency boundaries, the primary JAR, target metadata,
source and Javadoc JARs and (for these testkits) compiled TeaVM mock-engine separation.
Config and Graphics use the same goal and retain their library-specific runtime tests.
Transactions retains its Failsafe fixture, which also
examines the consuming application's generated TeaVM bootstrap and output.
The integration fixture declares both application targets and exercises the
generated transaction interceptor on the JVM and in Chrome.

The CDI bootstrap uses the same arrangement, with its browser-output check
in the Hello application. Reports are written under `target/failsafe-reports`.
JVM and browser execution tests run alongside these checks.

Resources and Config exercise their in-memory pipelines in Chrome. Their
portable stages are consumed through callbacks; JVM code that needs to wait
uses `JvmFutures.toCompletableFuture(...)`.

Model's lifecycle browser tests are enabled through `-Pteavm-tests`
and run in CI with TeaVM's SLF4J integration. Geometry, Geo, Graphics and Scene
also provide browser test profiles. These tests cover the exercised paths;
they do not make JVM-only APIs or host bindings portable.
