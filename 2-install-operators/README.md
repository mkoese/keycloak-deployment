
# Insall Operators

## Install Grafana Operator

Apply Grafana `OperatorGroup`

```shell
oc apply -f grafana/ -n grafana-dev
```

## Install Keycloak Operator

Apply Keycloak `OperatorGroup` and `Subscription`

```shell
# Community version
oc apply -f keycloak/ -n sso-dev

# Supported version
oc apply -f rhsso/ -n sso-dev
```

## Install Kafka Operator

Apply Keycloak `OperatorGroup` and `Subscription`

```shell
oc apply -f kafka/ -n kafka-dev 
```

## Check Operator Status

After applying the `OperatorGroup` and `Subscription` an installation process will start. Check the phase of the installation with following commands:

```shell
oc get csv -n grafana-dev
```

After reaching the `Succeeded` Phase the operator `Pod`s should run with the Status `Running`

```shell
oc get pod -n grafana-dev
```