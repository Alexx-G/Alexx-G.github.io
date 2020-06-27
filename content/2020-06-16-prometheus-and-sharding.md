+++
title = "Prometheus at scale in a Kubernetes environment"
description = """
Scaling horizontally Prometheus in a Kubernetes cluster might not be as easy
as you think. And here's why"""
+++

While Prometheus can handle perfectly fine even huge load (hundreds MBs of metrics scraped every 30s),
you might want to make sure that you can scale it horizontally at any moment.
And it might not be as easy as you think in a K8S cluster.

## A typical Prometheus deployment

Most Prometheus deployments in a K8S clusters are done just using [prometheus-operator](https://hub.helm.sh/charts/stable/prometheus-operator) Helm chart, which is a really convenient way to get it deployed.
The Helm chart has a few components that integrate nicely together, providing you with a ready-to-go monitoring stack:
- Prometheus
- CRDs for Prometheus-Operator
- Prometheus-Operator
- Grafana
- etc

However, it's quite easy to miss a quite important detail when using this setup: (by default) there's a **single** prometheus instance for the entire cluster.

This setup has some serious implications:
- Prometheus might easily hit your memory limits.
  It can be a huge problem in case Prometheus memory usage starts approaching memory limit of the available work nodes.
- In case Prometheus gets killed for some reason (eviction or restart loop due to liveness probe),
  you'll loose both alerting and monitoring for all services monitored by that Prometheus instance.

Both problems have the same solution - deploy several Prometheus instances and make
each Prometheus instance to scrape its own set of target. Additionally there might be a dedicate
Prometheus instance to scrape other Prometheuses in order to detect when one of them is offline.

## Divide et impera
