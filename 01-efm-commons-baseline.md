# Shared baseline and efm-commons

**Review date:** October 18, 2026  
**Scope:** Shared Maven foundation for DocuFlow and preparation for tenant `123`.  
**Status:** Architecture recommendations based on supplied source and screenshots. Runtime compatibility remains subject to devpod validation.

> This document proposes changes. No existing project code was modified, compiled, or run to produce it.

## 1. Architectural recommendation

Retain the current Maven parent hierarchy. Clarify artifact documentation, dependency ownership, and inherited build behavior before adding more shared modules.

Use `efm-commons-parent` as the shared build foundation. Keep application security, tenant configuration, database settings, application ZIP deployment, and business logic in DocuFlow.

## 2. Confirmed project hierarchy

The supplied POM screenshots and DocuFlow source show:

```text
com.ubs.AF2:flowable-ubs-addon-parent:3.17.16.18
└── com.ubs.efm:efm-commons-parent:1.0.0-SNAPSHOT
    ├── com.ubs.efm:efm-commons:1.0.0-SNAPSHOT
    │   └── Child JAR in the efm-commons-external directory
    │
    └── com.ubs.efm.docuflow:docuflow-workflow-parent:1.0.0-SNAPSHOT
        ├── docuflow-workflow-app
        └── docuflow-control-app [optional Maven profile]
```

This diagram represents Maven inheritance. DocuFlow is a separate consuming project; it is not shown as an aggregated module in the efm-commons root POM.

The `efm-commons` child JAR is not automatically a dependency of DocuFlow simply because both inherit from the same parent.

## 3. Baseline established by the evidence

| Component | Observed configuration | Assessment |
| --- | --- | --- |
| Corporate parent | `com.ubs.AF2:flowable-ubs-addon-parent:3.17.16.18` | Identified from the root POM screenshot |
| Shared parent | `com.ubs.efm:efm-commons-parent:1.0.0-SNAPSHOT` | Matches DocuFlow's parent reference |
| Shared child artifact | `com.ubs.efm:efm-commons`, packaging `jar` | Minimal Java module under `efm-commons-external` |
| Java | `java.version=17`; compiler release uses this property | Use JDK 17 consistently for the initial devpod baseline |
| Java enforcement | `[17,)` | Allows Java 17 and newer; does not prove compatibility with all newer JDKs |
| Flowable Platform BOM | `3.17.16` | Shared Flowable dependency baseline |
| Flowable Control in DocuFlow | `3.17.16` | Numerically aligned; check whether the Platform BOM manages its starter |
| Boot BOM in DocuFlow | `3.5.14` | Explicit downstream declaration; inherited corporate Boot configuration remains unknown |
| Springdoc | `2.8.15` | Managed by the shared parent |
| Spring Cloud Azure BOM | `5.23.0` | Version management does not itself add Azure libraries |

**Conclusion:** The baseline coordinates are identified. The complete effective dependency and plugin combination is not yet approved or runtime-verified.

## 4. Correct the artifact documentation first

The release-strategy screenshots say the repository publishes only `com.ubs.efm:efm-commons` with POM packaging and no child artifacts. The actual POM screenshots show:

| Artifact | Packaging | Purpose shown |
| --- | --- | --- |
| `efm-commons-parent` | `pom` | Shared Maven foundation |
| `efm-commons` | `jar` | Minimal Java application in `efm-commons-external` |

### Recommended changes

1. Update the README and release guide to describe the actual parent and child artifacts.
2. Keep DocuFlow's parent reference pointing to `efm-commons-parent`.
3. Document whether the external child JAR is intended for publication and consumption.
4. Distinguish repository directory names from Maven artifact IDs.
5. Document the intended release/version relationship between the parent and child.

Do not rename artifacts or change consumer coordinates solely to match the conflicting strategy document.

## 5. Define responsibility boundaries

| Layer | Recommended responsibility |
| --- | --- |
| Corporate addon parent | Approved platform baseline and corporate build requirements |
| `efm-commons-parent` | Shared EFM dependency management and build conventions |
| DocuFlow root POM | Module selection and DocuFlow-specific build decisions |
| Workflow application | Business APIs, tenant-aware execution, runtime configuration, application deployment |
| Control application | Administrative connection and Control runtime configuration |

Parent inheritance supplies Maven configuration. It does not automatically supply runtime tenant deployment, authentication, or authorization components.

The images do not establish that `efm-commons-external` implements any of those runtime capabilities.

## 6. Clarify the external module's role

The supplied child POM describes a minimal Java main-class module with a JAR manifest entry.

Choose and document its purpose:

| Intended purpose | Recommendation |
| --- | --- |
| Demonstration or baseline verification | Label it clearly as an example; do not add it as a DocuFlow runtime dependency |
| Reusable Java library | Define its public API, scope, consumers, and tests before consumption |
| Runtime Spring/Flowable extension | Establish its actual configuration and registration mechanism before claiming automatic integration |

For the current scope, retain the parent structure and avoid introducing additional BOM, starter, or runtime-library modules without a concrete need.

## 7. Establish dependency ownership

Prefer one intentional owner for each managed version. Avoid repeating the same version independently in corporate, shared, and application POMs.

### Proposed rules

- Inherit the approved corporate baseline where it meets DocuFlow's requirements.
- Put shared EFM adjustments in `efm-commons-parent` with a documented reason.
- Keep application-only exceptions in DocuFlow until they have a shared use case.
- Declare only the dependencies each application needs.
- Check whether the Flowable BOM already manages Control before maintaining an additional version declaration.

### Override register

The screenshots show explicit management for libraries including Netty, Nimbus, PDFBox, POI, and Azure components. Record why each override exists.

| Dependency or family | Version owner | Reason for override | Validated combination | Review trigger |
| --- | --- | --- | --- | --- |
| Netty | To confirm | To record | To record | Boot/Flowable baseline change |
| Nimbus JOSE JWT | To confirm | To record | To record | Security baseline change |
| PDFBox / POI | To confirm | To record | To record | Document-library baseline change |
| Azure libraries | To confirm | To record | To record | Azure/Boot BOM change |

Review related libraries together. An isolated `netty-handler` override, for example, may leave other Netty modules at different versions.

Dependency management controls versions when dependencies are used; it does not itself add those libraries to an application.

## 8. Make build conventions explicit

| Convention | Recommendation |
| --- | --- |
| Compiler | Retain Java 17 release and parameter metadata settings |
| Surefire | Manage the version centrally and keep application-specific test policies local |
| Source JAR | Verify activation; configuration under `pluginManagement` alone does not guarantee execution |
| Enforcer | Retain a deliberate Java requirement and document the chosen devpod JDK |
| JaCoCo | Verify the CI activation property and inherited execution behavior |
| Boot plugin | Verify effective version and executions before changing packaging |
| Resource filtering | Ensure tenant ZIP archives are copied without modification |
| SCM and release metadata | Ensure consuming projects retain appropriate repository-specific metadata |

Manage shared plugin defaults centrally, while activating executions deliberately. Keep application main classes in their respective application modules.

## 9. Outstanding Boot decision

DocuFlow explicitly imports Boot dependencies at `3.5.14`, but the screenshots do not show the corporate parent's Boot dependency or Maven plugin configuration.

Importing the Boot dependency BOM does not import Boot plugin management. Therefore, the application's Boot library and plugin versions must be checked separately.

### Evidence needed

- Corporate parent's parent declaration.
- Boot-related properties and dependency management.
- Boot Maven plugin management and active declarations.
- Plugin execution IDs, phases, and goals.
- Effective `mainClass`, `classifier`, `layout`, and `skip` configuration.
- Active profiles and resource filtering.

### Decision approach

| Finding | Proposed action |
| --- | --- |
| Corporate Boot dependencies and plugin are approved and aligned | Inherit them; remove redundant declarations after checking equivalence |
| Plugin management is missing | Add it at the appropriate shared layer |
| DocuFlow requires an intentional Boot override | Validate the complete combination with Flowable and corporate extensions |
| Overlapping management produces mixed versions | Resolve ownership before implementation |

Do not retain or remove `3.5.14` unconditionally until the inherited configuration is available.

## 10. Versioning and reproducibility

For the initial snapshot evaluation:

1. Record the exact efm-commons source revision or resolved snapshot identity.
2. Record the corporate parent coordinate and effective dependency/plugin baseline.
3. Keep changes small enough to compare against the previous baseline.
4. Publish and consume a stable parent release when available and validated.

The screenshots do not establish that a stable parent release has already been published.

## 11. Suggested implementation order

| Priority | Action | Completion evidence |
| --- | --- | --- |
| 1 | Correct artifact documentation | README, release guide, and actual POMs agree |
| 2 | Review inherited Boot configuration | Effective dependency/plugin versions and origins recorded |
| 3 | Resolve version ownership | Duplicate declarations removed only where verified redundant |
| 4 | Verify plugin activation and packaging | Intended executions and artifact outputs documented |
| 5 | Record a reproducible baseline | JDK, Maven, parent revision, and dependency combination captured |
| 6 | Apply the agreed baseline to both DocuFlow applications | Workflow and Control are evaluated against the same approved foundation |

## 12. Devpod acceptance checklist

- [ ] Artifact coordinates and packaging agree with the documentation.
- [ ] The purpose of `efm-commons-external` is documented.
- [ ] The devpod JDK and Maven versions are recorded.
- [ ] Effective Boot dependency and plugin versions are reviewed.
- [ ] Flowable Platform and Control versions are intentionally aligned.
- [ ] Dependency overrides have recorded reasons.
- [ ] Plugin execution behavior is understood for both applications.
- [ ] Tenant ZIP resources remain unmodified during packaging.
- [ ] The shared parent snapshot revision or stable release is recorded.

## References

- [Maven dependency management](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
- [Maven POM reference: inheritance and plugin management](https://maven.apache.org/pom.html)
- [Spring Boot Maven plugin: using a corporate parent](https://docs.spring.io/spring-boot/3.5/maven-plugin/using.html)

The supplied project source and screenshots are the evidence for internal artifact coordinates. Public documentation explains Maven and Boot behavior; it does not establish compatibility of the internal corporate baseline.
