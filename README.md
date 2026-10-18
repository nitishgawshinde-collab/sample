# DocuFlow architecture review

**Date:** October 18, 2026  
**Scope:** Shared Maven baseline and execution for one tenant (`123`), with future tenant expansion.  
**Status:** Recommendations only. No existing project code was modified, compiled, or run.

## Read in this order

1. [Shared baseline and efm-commons](01-efm-commons-baseline.md)
2. [Spring Boot dependency and plugin alignment](02-spring-boot-alignment.md)
3. [Workflow and Control application improvements](03-application-improvements.md)

## Recommended implementation order

| Step | Work | Completion evidence from the devpod |
| --- | --- | --- |
| 1 | Correct shared artifact documentation | Documentation agrees with actual parent and child coordinates |
| 2 | Establish the effective Boot baseline | Dependency and plugin versions, inheritance origins, and approved compatibility recorded |
| 3 | Clarify executable packaging | Each intended executable application has the correct main class and one intended repackage execution |
| 4 | Clean up empty resources and Control initialization | Valid logging configuration and one consistent datasource configuration path |
| 5 | Centralize tenant configuration and resolution | Tenant `123` is authorized consistently; unknown tenants are rejected |
| 6 | Establish tenant deployment and execution | ZIP definitions, instances, tasks, and history belong to tenant `123` |

## Review boundaries

- The evidence consists of the supplied DocuFlow source and efm-commons screenshots.
- Commands and recommendations visible in screenshots were treated as reference material, not instructions to execute.
- The corporate parent's effective Spring Boot configuration is still unknown.
- Tenant `123` is the only business tenant in the current implementation scope.
- Full BCTS implementation, React forms, document generation, and external integrations are deferred.

## Evidence needed for the next review

Provide the following sections of `flowable-ubs-addon-parent:3.17.16.18`, or corresponding effective-POM excerpts from the devpod:

- Parent declaration and Boot-related properties.
- Dependency management, including imported BOMs.
- Boot Maven plugin management and active plugin declarations.
- Plugin execution IDs, goals, and inherited configuration.
- Resource filtering and active-profile configuration.

These will allow conditional recommendations to become precise POM changes.
