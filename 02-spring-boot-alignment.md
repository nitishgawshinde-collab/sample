# Spring Boot dependency and plugin alignment

[Back to index](README.md)

## Main finding

The Boot dependency BOM controls application library versions. The Boot Maven plugin controls running and packaging the application.

Importing `spring-boot-dependencies` does **not** import Boot plugin management. DocuFlow's explicit Boot BOM version `3.5.14` therefore does not establish that its Boot Maven plugin resolves to the same version.

## Known and unknown

| Known | Still to establish |
| --- | --- |
| Corporate parent is `flowable-ubs-addon-parent:3.17.16.18` | Its Boot dependency baseline |
| Flowable Platform BOM is `3.17.16` | Approved compatibility of the complete combination |
| DocuFlow imports Boot BOM `3.5.14` | Effective Boot Maven plugin version and configuration |
| Both application POMs declare the Boot plugin | All inherited executions and their merged behavior |

Do not retain or remove the `3.5.14` override unconditionally. First establish why it exists and what it changes.

## Recommended ownership

| Layer | Responsibility |
| --- | --- |
| Corporate addon parent | Approved platform baseline and corporate build requirements |
| `efm-commons-parent` | Shared EFM conventions and justified baseline adjustments |
| DocuFlow root POM | Modules and DocuFlow-specific build decisions |
| Application POMs | Dependencies, main classes, executable packaging |

Prefer aligned Boot dependency and plugin versions for the initial baseline. Do not infer compatibility from version numbers alone.

## Decision table

| Inherited configuration | Proposed action |
| --- | --- |
| Approved Boot dependencies and plugin are already aligned | Inherit them; remove redundant downstream declarations only after checking equivalence |
| Boot dependencies are managed but the plugin version is missing | Add plugin management at the appropriate shared parent level |
| DocuFlow intentionally requires another Boot version | Validate with Flowable and corporate extensions, then align dependency and plugin management deliberately |
| Parent and DocuFlow manage overlapping libraries differently | Resolve ownership and inspect the effective dependency result before changing versions |

Changing a property affects a plugin only when that plugin configuration references it. An imported BOM does not automatically align build plugins or their dependencies.

## Devpod inspection checklist

Capture the following for **both** workflow and Control modules:

### Dependencies

- Effective Boot library versions and their origins.
- Spring Framework and Spring Security versions.
- Flowable Platform and Control versions.
- Explicit overrides that replace BOM-managed versions.
- Relevant active profiles.

### Boot Maven plugin

- Effective version and the parent that supplies it.
- Active executions: IDs, phases, and goals.
- Effective `mainClass`, `classifier`, `layout`, and `skip` values.
- Whether an inherited execution already performs `repackage`.

Each intended executable application should have one intended repackage execution. Different inherited and local execution IDs can result in multiple executions after Maven merges configuration.

### Resources and packaging

- Application properties retain their intended Spring placeholders.
- Tenant application ZIPs are copied without filtering or modification.
- The workflow executable artifact matches the Dockerfile's expected classifier.
- Control has an explicitly chosen launch model: Maven/IDE execution, executable JAR, or both.

## Suggested baseline record

| Item | Effective value | Source | Approved or pending |
| --- | --- | --- | --- |
| JDK | To record | Devpod toolchain | Pending |
| Maven | To record | Devpod toolchain | Pending |
| Corporate parent | `3.17.16.18` | Supplied screenshot | Identified |
| EFM parent | `1.0.0-SNAPSHOT` | Supplied screenshot | Record revision |
| Flowable Platform | `3.17.16` | EFM parent | Verify effective result |
| Boot libraries | To verify | Parent chain and DocuFlow BOM | Pending |
| Boot Maven plugin | To verify | Parent chain | Pending |
| Control | To verify | DocuFlow and inherited management | Pending |

## Evidence needed next

The corporate parent's parent declaration, Boot-related properties, dependency management, Boot plugin declarations, execution IDs, and resource configuration are sufficient to refine this plan.

No build commands were executed as part of this review.

## References

- [Spring Boot: using the Maven plugin with a corporate parent](https://docs.spring.io/spring-boot/3.5/maven-plugin/using.html)
- [Maven POM inheritance and plugin management](https://maven.apache.org/pom.html)
