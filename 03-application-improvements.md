# Workflow and Control application improvements

[Back to index](README.md)

## Scope

Establish a clean runtime foundation and execution for business tenant `123`. Additional tenants should be possible later without scattering tenant-specific logic through Java code.

Keep the two existing application modules. Additional business modules and full workflow features are outside this phase.

## Workflow application

### Suggested minimum package structure

```text
com.ubs.efm.docuflow
├── api/
├── application/
│   └── WorkflowApplicationService
├── tenancy/
│   ├── TenantProperties
│   └── TenantResolver
├── workflow/
│   └── deployment/       only if custom deployment is required
├── security/
└── configuration/
```

These class names are proposals, not implemented changes.

### Priority changes

| Change | Purpose |
| --- | --- |
| Move engine calls behind an application service | Centralize authorization, validation, and tenant-aware execution |
| Centralize enabled-tenant configuration | Keep `123` out of controllers and delegates |
| Resolve and authorize the tenant from identity | Prevent callers from selecting an unauthorized tenant |
| Define one ZIP deployment mechanism | Avoid duplicate deployment or inconsistent tenant assignment |
| Copy application ZIPs without resource filtering | Preserve the exported archive |
| Remove the unused empty sample BPMN | Tenant ZIPs provide the application models |
| Replace or remove empty logging placeholders | Ensure logging configuration is valid |
| Introduce a devpod profile when needed | Separate devpod networking, authentication, and datasource settings from local development |

### Tenant execution contract

Use `123` as a string identifier consistently.

1. Authenticate the caller.
2. Resolve the tenant through a central component.
3. Check that the tenant is enabled and authorized for that caller.
4. Pass the tenant explicitly into workflow operations.
5. Check tenant ownership and permissions for tasks and history.
6. Reject unknown tenants rather than silently falling back to `123`.

The existing local `permitAll()` chain can support a connectivity experiment, but it does not establish authenticated tenant execution.

### Application ZIP placement

```text
src/main/resources/com/flowable/apps/123/
└── <exported-application>.zip
```

This project convention needs explicit resource discovery and tenant targeting. A folder named `123` alone does not establish deployment ownership.

Prefer supported Flowable configuration or verified corporate deployment functionality before adding a custom scanner. The shown efm-commons POM does not prove that a runtime tenant deployer exists.

When more tenants are introduced, use explicit package-to-tenant mappings. Do not broadly scan all tenant packages and deploy them to every configured tenant.

## Control application

Keep Control as a thin administrative application with its own metadata schema.

| Area | Suggested improvement |
| --- | --- |
| Maven inclusion | Retain the opt-in `local-control-app` profile |
| Packaging | Decide whether an executable JAR is required and verify the inherited repackage execution |
| Main class | Keep startup focused on Spring application bootstrap |
| Schema preparation | Move preliminary JDBC schema creation into documented provisioning or correctly ordered Spring-managed initialization |
| Datasource configuration | Use one consistent property source instead of reading environment variables separately before Spring starts |
| Credentials | Support dedicated Control database credentials, even if the initial devpod reuses an account |
| Security customization | Remove the empty customization class unless actual configuration will be added |
| Backend connection | Document target URL, authentication, and administrative scope |

The current pre-Spring JDBC connection cannot honor all Spring configuration sources, such as command-line datasource overrides. That is the main reason to change the initialization approach.

The Maven profile controls build inclusion. Network access and authentication are separate runtime concerns.

## Database boundary

- Keep workflow and Control tables in separate schemas.
- Use tenant identifiers within the workflow schema for the initial design.
- Do not introduce a database or schema per tenant solely for future flexibility.
- Define schema preparation as part of the devpod setup.

## Devpod acceptance checklist

- [ ] Workflow and Control use the intended effective dependency/plugin baseline.
- [ ] Valid logging configuration is present.
- [ ] Required schemas exist and datasource settings have one consistent source.
- [ ] Tenant ZIP definitions deploy under `123`.
- [ ] An authenticated tenant-123 user can start the intended process.
- [ ] Instances, tasks, and history retain tenant ownership.
- [ ] Unconfigured or unauthorized tenant selections are rejected.
- [ ] Restart with an unchanged ZIP does not create unintended deployments.
- [ ] Control can inspect the deployment and instance under tenant `123`.

Full isolation between two active tenants remains a later validation step before onboarding the second tenant.

## References

- [Flowable multitenancy](https://documentation.flowable.com/latest/admin/installs/mt)
- [Flowable application auto-deployment](https://documentation.flowable.com/latest/develop/be/auto-deployment)

Public documentation describes the supported concepts. Confirm exact behavior against Flowable `3.17.16` and your corporate extensions during devpod validation.
