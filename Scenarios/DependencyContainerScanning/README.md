# Dependency and Container Scanning Scenarios

When selecting third-party components (both commercial and open source), it’s important to understand the impact that a vulnerability in the component could have on the overall security of the system. Software Composition Analysis tools can assist with licensing exposure, provide an accurate inventory of components, and report any vulnerabilities with referenced components. This applies to dependencies at both component and container level.

## Dependency Scanning

- [Aqua Dependency scanning](./Aqua.md)
- [Dependency Scanning with Sonar Qube](./sonarQube-Dependency.md)
- [Dependency Scanning with WhiteSource](./WhiteSource-Dependency.md) ([Account Setup Required](./WhiteSource-Setup.md))
- [Sample Whitesource Pipeline](../../pipelines/DependencyScanning/WhiteSource.yml)

## Container Scanning

- [Aqua Container scanning](./Aqua.md)
- [Sample Aqua yml pipeline](../../pipelines/ContainerScanning/Aqua-CI.yml)
- [Container Scanning with WhiteSource](./WhiteSource-ContainerScanning.md) ([Account Setup Required](./WhiteSource-Setup.md))

## License Scanning

- [OSS License scanning with FOSSA](./FOSSA.md)
