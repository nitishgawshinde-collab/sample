# Plan: incorporate platform-build into EFM and DocuFlow

**Date:** 19 October 2026  
**Scope:** Shared build baseline, quality checks, and preparation for tenant `123`.  
**Status:** Proposed architecture; implementation and validation belong in your devpod.

No existing project code was changed, compiled, or run. Commands and migration instructions inside the supplied package were treated as reference material, not instructions to execute.

## 1. Decision

**Use both repositories, with clear ownership.** Adapt the reusable quality configuration into `efm-commons`; activate and configure it for the applications in `docuflow-workflow`.

Do not adopt the supplied `workflow-platform-parent` as DocuFlow's replacement parent. It directly inherits Spring Boot `3.5.12`, while your existing hierarchy inherits the corporate Flowable addon parent. Replacing that hierarchy could discard corporate dependency management, plugin executions, repositories, and packaging conventions.

The supplied package is a useful starting point for build tooling. It is not a verified replacement for your Enterprise Flowable baseline or a tenant execution framework.

## 2. Evidence and limits

Reviewed material includes the supplied `platform-build` POMs, quality rules, templates, migration notes, and sample application; the DocuFlow source; and your earlier EFM screenshots.

| Evidence | Implication |
| --- | --- |
| EFM inherits `flowable-ubs-addon-parent:3.17.16.18` and manages Flowable Platform `3.17.16` | Preserve this inheritance while introducing quality tooling |
| DocuFlow explicitly imports Boot BOM `3.5.14` | Resolve its relationship with inherited Boot configuration before changing it |
| Supplied parent inherits Boot `3.5.12`; Enterprise BOM is only a commented placeholder | It does not reproduce your Enterprise platform baseline |
| Sample uses OSS Flowable `7.1.0`, H2, and JPA | Do not copy its dependency stack into DocuFlow |
| Package README describes tool checks with framework stubs, without a complete Maven verification | Full dependency resolution and build compatibility remain unproven |
| EFM source and corporate parent effective configuration were not supplied directly | Their complete inheritance and merge behavior remain devpod verification items |

Treat comments claiming a certified Boot/Flowable combination as claims requiring confirmation against your internal supported baseline.

## 3. Ownership map

| Supplied material | Recommended destination | Treatment |
| --- | --- | --- |
| `quality-config` resource JAR | New `efm-quality-config` module in EFM | Reuse after reviewing rules and suppressions |
| Quality plugin versions and common configuration | `efm-commons-parent` plugin management | Merge selectively with existing corporate/EFM configuration |
| Compiler and Enforcer settings | Existing EFM configurations | Reconcile; avoid a second competing configuration |
| JaCoCo setup | Existing EFM JaCoCo configuration | Consolidate executions and define coverage ownership |
| Direct Boot parent declaration | No destination | Retain the corporate parent chain |
| Placeholder Enterprise BOM and sample OSS dependencies | No destination | Keep the existing approved Enterprise dependencies |
| Quality plugin activation | DocuFlow application POMs | Explicit opt-in, initially through a named profile |
| Local suppressions and module coverage thresholds | DocuFlow repository | Keep exceptions near the affected application |
| `.editorconfig`, `.mvn/maven.config`, `lombok.config` | DocuFlow repository root | Review and adapt individually |
| GitHub PR template | GitLab merge-request template, if applicable | Translate to the actual repository workflow |
| Example Java application | Reference only | Select coding practices, not domain or persistence design |
| Example deliberately bad code | Tooling verification fixtures | Never include in an application source set |
| Nexus URLs and `com.yourorg` coordinates | Replace during implementation | Use approved internal publishing settings; credentials stay in Maven settings/CI secrets |

## 4. Proposed structure

```text
efm-commons/
  pom.xml                         # existing efm-commons-parent
  efm-commons-external/            # existing module; no new runtime role
  efm-quality-config/             # proposed resources-only build artifact
    pom.xml
    src/main/resources/quality/
      checkstyle.xml
      checkstyle-suppressions.xml
      pmd-ruleset.xml
      spotbugs-exclude.xml
  docs/
    quality-adoption.md

docuflow-workflow/
  pom.xml                         # existing module selection and app baseline
  .mvn/maven.config               # reviewed shared Maven options
  .editorconfig
  lombok.config                   # only if its conventions are needed
  config/quality/
    checkstyle-suppressions.xml
  .gitlab/merge_request_templates/
    Default.md                    # adapt to existing GitLab conventions
  docuflow-workflow-app/
    pom.xml                       # explicit quality activation
    src/main/resources/com/flowable/apps/123/
      <exported-tenant-app>.zip
  docuflow-control-app/
    pom.xml                       # separate activation and coverage baseline
```

The new rules JAR has a concrete reuse purpose, unlike adding an unspecified shared runtime library. It belongs on the quality plugins' dependency classpaths, not in application `<dependencies>`. Keep it separate from the existing `efm-commons` JAR.

Avoid a bootstrap cycle: the rules artifact must build without resolving itself through an inherited quality plugin. A minimal standalone POM, aggregated by EFM but not inheriting application quality executions, is a straightforward option. It needs no Boot repackaging or application coverage gate. Publish the rules artifact before publishing a parent version that requires it.

## 5. Integrating with inherited Boot configuration

Keep this inheritance:

```text
flowable-ubs-addon-parent:3.17.16.18
  -> efm-commons-parent
     -> docuflow-workflow-parent
        -> workflow-app / control-app
```

Before adjusting Boot, capture the effective configuration in the devpod for both applications, including the `local-control-app` profile. Record:

1. Resolved Boot dependency versions and the BOM responsible for them.
2. Resolved `spring-boot-maven-plugin` version and its configuration source.
3. All `repackage` execution IDs, phases, classifier, layout, main class, and skip settings.
4. Corporate resource filtering and packaging executions affecting application ZIPs.
5. Compiler, Surefire, Enforcer, and JaCoCo versions and merged executions.

A Boot dependency BOM does not provide Boot plugin configuration. Likewise, importing a POM as a BOM does not import its build plugins. Maven documents the distinction in its [POM reference](https://maven.apache.org/pom.html).

For Workflow, preserve the intended `exec` classifier and `ZIP` layout until their consumer/deployment requirements are confirmed. Check for duplicate repackaging caused by inherited and local execution IDs. For Control, a local `mainClass` declaration alone does not establish whether repackaging is bound to the lifecycle; inspect inheritance.

Do not change Boot to `3.5.12` simply because the supplied template uses it. Do not remove DocuFlow's `3.5.14` BOM until the corporate baseline is known. If an override is necessary, document and validate the approved dependency/plugin combination in one owning layer.

The supplied parent also adds SpotBugs annotations as an inherited `provided` dependency. Prefer declaring annotation dependencies only where used. Do not assume `provided` guarantees their absence from executable Boot archives; verify the packaged artifact and configure exclusions where required. See [Spring Boot archive packaging](https://docs.spring.io/spring-boot/maven-plugin/packaging.html).

## 6. Changes needed in the supplied quality design

### Explicit activation

The template activates a parent profile using `${basedir}/src`. Do not rely on that to activate checks independently in every consuming application. Maven resolves profiles early; profile definitions themselves are not inherited, while active effects are. See [Maven profile inheritance](https://maven.apache.org/guides/introduction/introduction-to-profiles.html).

Recommended initial design: common configuration in EFM `pluginManagement`, with an explicit `quality` profile in each DocuFlow application POM declaring the selected plugins. CI enables that profile deliberately. Keep aggregation-only and resource-only modules outside Java analysis and coverage execution.

Managed configuration can still affect plugins already active through inheritance. Review effective configurations even when new tools are opt-in; Enforcer and compiler are particularly relevant.

### One JaCoCo configuration

EFM screenshots already show JaCoCo `0.8.8`, a `ci-build` profile, and executions named `jacoco-initialize` and `post-unit-test`. The supplied template proposes `0.8.13` with additional prepare-agent, report, and check executions.

Choose one validated version and consolidate the execution set. Preserve existing test JVM arguments when attaching the agent. Establish whether unit and integration coverage are reported separately or merged before setting gates. Do not indiscriminately exclude `*Application`: Control's main class currently contains schema initialization behavior that deserves verification.

### Honest rollout controls

`quality.failOnViolation=false` is not a universal report-only mode. Spotless, compiler warnings, Enforcer, and coverage gates have separate settings. Define report-only behavior tool by tool, while keeping tool execution/configuration failures visible.

Start with reports and a measured baseline. The supplied 80% line / 70% branch thresholds are targets, not evidence of current coverage. Existing empty test files do not establish tested behavior. Give Workflow and Control explicit, justified module thresholds once real tests exist.

Spotless ratcheting limits formatting scope; it does not automatically enforce “new findings only” for PMD, Checkstyle, or SpotBugs. A branch comparison also requires the chosen Git reference to exist in the devpod/CI checkout.

### Review rule exclusions

Review broad suppressions before sharing them across teams. Examples include SpotBugs exclusions based on `Service`/`Controller` class suffixes, global constructor exceptions, generated-looking paths, and PMD suppression around `BpmnError` construction. Prefer narrow exceptions with a reason, owner, and review date.

Validate that expected violations are detected and intended exceptions are excluded using small fixtures. A passing sample with extensive suppressions is insufficient evidence. Regex checks for secrets or placeholders complement, but do not replace, dedicated scanning and review.

Do not apply a blanket ban on every `javax` package: Java SE APIs such as `javax.sql` remain legitimate. Review specific legacy dependency coordinates and their actual namespaces against the approved platform.

### Version and path validation

Treat the supplied analyzer/plugin versions as candidates until resolved and exercised on JDK 17 in the devpod. Verify plugin-engine compatibility, rule loading from the rules JAR, formatter version, and internal repository availability. Do not introduce the supplied Java 21 profile in this first adoption.

The local suppression path uses `${maven.multiModuleProjectDirectory}`. Verify it with root builds, module-selected builds, and IDE builds. Document the supported invocation location and retain a repository-root `.mvn` directory if relying on this mechanism.

## 7. Application-specific boundaries

**Workflow application:** apply formatting, Java analysis, and tests to the existing implementation. Keep tenant resolution, authorized tenant access, ZIP deployment, Flowable calls, and application security in DocuFlow. These are runtime responsibilities, not quality-rule responsibilities.

For tenant `123`, the later implementation should demonstrate explicit tenant assignment at deployment and execution, rejection of unsupported tenant values, and tenant-scoped access to tasks/history. The directory name alone does not prove those behaviors. Static quality checks cannot establish tenant isolation; targeted integration tests are required in the devpod.

**Control application:** reuse relevant Java checks but keep its configuration and tests separate. Its inclusion remains controlled by `local-control-app`. Validate its packaging in an explicit CI job/profile without accidentally making Control a mandatory runtime deployment alongside Workflow.

**Sample code:** constructor injection, thin controllers, DTOs, and testable delegates can inform implementation. Do not copy tenantless process starts, unscoped repository access, stub risk clients, or new JPA entities into this scope. A mock test does not prove transaction coordination between JPA and Flowable. Converting technical failures into `BpmnError` also requires an explicit workflow error/retry policy.

## 8. Ordered devpod implementation plan

| Phase | Work | Completion evidence |
| --- | --- | --- |
| 1. Baseline | Capture current effective POMs, active profiles, dependency trees, and packaging expectations | Corporate/EFM/DocuFlow ownership of Boot and plugins documented |
| 2. Rules artifact | Adapt coordinates, review rules, publish an isolated `efm-quality-config` candidate | Rules JAR contains expected resources and resolves without self-dependency |
| 3. Shared management | Merge plugin defaults into an EFM candidate version; reconcile existing executions | No unintended dependency or packaging changes in consumers |
| 4. Workflow pilot | Explicitly activate tools in report mode; collect findings | Every selected tool actually runs; execution failures remain visible |
| 5. Meaningful checks | Add tenant-focused tests and fix prioritized findings; enable gates incrementally | Deliberately introduced violations fail; accepted exceptions are narrow |
| 6. Control pilot | Activate the optional module and apply relevant checks separately | Control packaging and behavior verified with its own test baseline |
| 7. Stable release | Publish tested rules/parent versions and pin DocuFlow to them | Reproducible pipeline with documented rollback coordinates |

Use devpod Maven settings for internal artifact access. Effective-POM generation, dependency resolution, compilation, tests, and runtime checks are future devpod activities; none were executed for this review.

Avoid introducing formatting changes and behavioral changes in the same implementation commit. Shared parent changes can affect all inheriting applications, particularly consumers following snapshots. Pilot with explicit candidate coordinates before a wider release.

## 9. Acceptance criteria and rollback

- Corporate parent inheritance and Enterprise Flowable dependencies remain intentional and documented.
- Workflow and Control each have the expected Boot packaging execution, without accidental duplicates.
- Rules resolve through plugin dependencies and are absent from application runtime dependencies.
- Quality execution is visible in CI for the intended modules; missing activation cannot masquerade as success.
- Existing compiler, Enforcer, test JVM arguments, and JaCoCo behavior are reconciled.
- Workflow ZIP resources are retained unmodified in packaging; tenant deployment is separately verified.
- Thresholds reflect meaningful tests; suppressions do not hide representative defects.
- Previous parent and rules versions are recorded for rollback; reverting tooling does not require runtime code changes.

## 10. Recommended first implementation slice

Start with **the inherited baseline audit, a small shared rules artifact, and explicit Workflow quality activation**. Introduce formatter/Checkstyle first, then PMD and SpotBugs, and finally coverage enforcement after meaningful tests exist. Existing mandatory corporate checks should continue throughout.

This gives DocuFlow a controlled adoption path and leaves EFM reusable for future workflow applications without introducing a second competing parent hierarchy.
