# Python service for Kubernetes on Wodby

Build and run Python applications on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Python.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Start with a boilerplate

Use one of the boilerplates exposed by this service to start with compatible
build configuration and Wodby CI:

- [Python boilerplate](https://github.com/wodby/python-boilerplate)

## Wodby stacks using this service

- [Python application stack](https://github.com/wodby/stack-python)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `python` |
| Type | Application service |
| Versions | `3.14` by default; also available: `3.13`, `3.12`, `3.11`, `3.10` |
| Workloads | `main` (Deployment, primary) |
| Containers | `python` using `wodby/python`, build target |
| Endpoints | `python`: HTTP 8080 (main) |
| Service links | DBMS (`db`), optional; Mail Transfer Agent (`sendmail`), optional; Redis (`redis`), optional |
| Application build | Git source connection enabled; Dockerfile: `Dockerfile`; boilerplates: [Python boilerplate](https://github.com/wodby/python-boilerplate) |
| Helm | chart `oci://registry-1.docker.io/wodby/python`; version `0.1.0` |
| Configuration and operations | 1 integration slots |

## Use this service

Use this service through [Python application stack](https://github.com/wodby/stack-python), or reference `python` from a
custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
