# Prepare OpenShift Culster for deployment

Create all required `projects`

```shell
oc new-project sso-dev
oc new-project grafana-dev
oc new-project kafka-dev
oc new-project app-dev
```

Enable user workload monitoring for the internal Prometheus

```shell
oc apply -f cluster-monitoring-config.yml
```

