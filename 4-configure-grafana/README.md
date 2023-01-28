# Configure Grafana with Dashboards and Datasources

## Grafana instance configuration

After executing this command, a new instance of Grafana will be generated and made available for use.

```shell
oc apply -f cr-grafana.yaml -n grafana-dev
```

This command will retrieve the Grafana route and extract the host field from the returned JSON output. The host field contains the hostname for the Grafana route. In a short while, the Grafana instance should be accessible via this URL.

```
GRAFANA_URL=https://$(oc get route grafana-route -n grafana-dev -o jsonpath='{.spec.host}' -n grafana-dev)
echo $GRAFANA_URL
```

The command below will store the user credentials for Grafana in environment variables. To access the Grafana dashboard, you can then use the "echo" command to display the values of the environment variables.

```shell
GRAFANA_ADMIN=$(oc get secret grafana-admin-credentials -o jsonpath='{.data.GF_SECURITY_ADMIN_USER}' -n grafana-dev | base64 --decode)
GRAFANA_PASSWORD=$(oc get secret grafana-admin-credentials -o jsonpath='{.data.GF_SECURITY_ADMIN_PASSWORD}' -n grafana-dev | base64 --decode)
```

```shell
echo $GRAFANA_ADMIN
echo $GRAFANA_PASSWORD
```

## Grafana `ServiceAccount` configuration

In order to allow the service accounts associated with Grafana to access the Prometheus namespace, it is necessary to create a ClusterRoleBinding. This will enable the Grafana service accounts to have the appropriate level of access to the Prometheus namespace, allowing them to retrieve the necessary metrics for displaying in Grafana.

```shell
oc apply -f sa-grafana-rolebinding.yaml -n grafana-dev
```

With the next command, we will save the token for the service account which will be used for accessing the grafana service. This will enable us to authenticate to the Prometheus service using the service account's token.

```shell
GRAFANA_SA_TOKEN=$(oc sa new-token grafana-serviceaccount -n grafana-dev)
```

```shell
echo $GRAFANA_SA_TOKEN
```

To replace the environment variable $GRAFANA_SA_TOKEN in the file cr-grafana-datasource.yaml with its corresponding value, please run the following GNU sed command:

```shell
sed -i "s/\$GRAFANA_SA_TOKEN/$GRAFANA_SA_TOKEN/g" cr-grafana-datasource.yaml
```

Please make sure that the environment variable $GRAFANA_SA_TOKEN has been correctly replaced in the yaml file before running the next command.

Use the following command to apply the modified datasource cr yaml file with the oc apply command:

```shell
oc apply -f cr-grafana-datasource.yaml
```