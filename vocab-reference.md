# List of Spinnaker Concepts and Terms

Jump to [TOC](/README.md)

---

**Instance:** a running container or VM

**Server Group:** a group of identical running instances in one region and named following the Spinnaker naming convention _application-stack-detail-version_

**Cluster:** a collection of server groups, in different regions or with different versions, grouped by the naming convention _application-stack-detail_

**Application:** A grouping of clusters, grouped by the _application_ portion of the naming convention

**Security Group:** a set of firewall rules defined by an IP range (CIDR) along with a communication protocol and a port range

**Load Balancer:** associated with an ingress protocol and port range, balances traffic between instances in server groups and optionally provides healthchecks. In Kubernetes, these are services.

**Naming Convention/Frigga:** the library Spinnaker uses to parse names. _application-stack-detail-version_ convention

**Stack:** a user-defined word to indicate a logical grouping for a cluster (such as "prod" or "staging" or "test")

**Detail:** any following user-defined word to indicate a secondary logical grouping for a cluster

**Strategy/Pipeline Strategy:** the way that a new server group will be deployed. Options include:

* _None_: the server group is deployed
* _Red/Black_: the server group is deployed and then the previous server group is disabled
* _Highlander_: the server group is deployed and then all other server groups are destroyed
* _Custom_: you can create your own strategy, using a pipeline. Outside the scope of this workshop
