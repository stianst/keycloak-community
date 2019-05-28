# Keycloak.Next

* **Status**: Research
* **Author**: [stianst](https://github.com/stianst)
* **JIRA**: [KEYCLOAK-10213](https://issues.jboss.org/browse/KEYCLOAK-10213)

## What are we trying to improve?

### Storage

* Database schema is complex
* Caching layer is very complex and brittle
* Zero downtime upgrades is more or less impossible
* Cross DC support is complex
* Sessions can't be persisted
* Realm scalability issues
* Client scalability issues
* Multi-tenancy issues

### Distribution

* Most focus on ZIPs, containers are secondary
* Need to test multiple OS as well as JDKs
* High memory footprint
* High startup time

### Configuration management

* Hard to manage configuration
* Hard to deal with Keycloak in a CI/CD environment 

### Providers

* Java only support
* Sand-boxing of custom code in multi-tenant environments 
* Versioning and deprecation of APIs


## Migration to Keycloak.Next

The aim is to have as few breaking changes as possible, but we will not limit ourselves with this regards. There will
be a fairly big impact on users to migrate to Keycloak.Next. For the storage we need to develop tooling to do this
migration automatically.

## Timing

* 2019 - A working prototype
* 2020 - Production ready, deprecate legacy distribution
* 2021 - Drop legacy distribution  
