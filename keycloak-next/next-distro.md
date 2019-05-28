# Keycloak.Next Distro

Keycloak is currently built on-top of WildFly. WildFly is a full JEE container, but Keycloak is not a JEE application.
Keycloak uses JAX-RS and Hibernate APIs only and does not have a direct requirement on JEE beyond these specifications.

There are both pros and cons to using WildFly. Some pros include a well tested container that provides a number of
features needed like configuration, datasources and dependencies. Some cons include high startup time and memory
footprint, as well as including a large number of dependencies and features that Keycloak does not need.

Quarkus is a project that was recently introduced to address issues with using Java based applications in the cloud. It
focuses on being a lightweight framework to build cloud native micro-services using Java.

By moving to Quarkus we will achieve much lower startup and memory footprint (comparable to Go based projects). It's not
a trivial change though and will require a fair bit of work to get it to work.  