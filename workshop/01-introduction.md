## Introduction
DevOps is a term we hear tossed around daily. It encompasses the tools and practices that break down silos between teams to help them create, deliver, and deploy applications and services faster than ever before. Previously organizations only released a few times a year. Today under the prevailing Continuous Delivery model, they can push out releases in a matter of weeks or days.

Blue green deployment is an application release model that gradually transfers user traffic from a previous version of an application or microservice to a nearly identical new release. The fundamental idea is to have two easily switchable environments to switch between. To transfer the traffic between two different version we will need to split the traffic.

In today's workshop we will deploy HashiCorp Consul with the official Helm chart on pre-built Kubernetes cluster. After deploying Consul, you will learn how to access the Consul dashboard. After you will validate that Consul works you will deploy Chuck Norris App version 1 and version 2. We will start slowly migrating our application from version 1 to version 2 and monitor the whole migration with Grafana.

In this workshop you will have available pre-built `kind` (Kubernetes in Docker) Kubernetes cluster.

We will be covering following areas:
- Install Consul in recommended minimal configuration - Consul, Prometheus and Consul Ingress
- Install Grafana
- Validate your Consul and Grafana
- Deploy Chuck Norris Application
- Make Chuck Norris Application part of Consul Service Mesh
- Test the application
- Deploy Consul Service Resolver and Splitter
- Configure Consul Ingress
- Steer traffic from Chuck Norris Application version 1 to version 2
