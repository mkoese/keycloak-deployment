# Configure Keycloak with extensions

## Keycloak instance configuration

With the command being executed, a new Keycloak instance will be created along with an extension called [Keycloak-Metrics-SPI](https://github.com/aerogear/keycloak-metrics-spi) which will allow for additional functionality related to metrics and performance measurement.

After executing this command, a new instance of Keycloak will be generated and made available for use.

```shell
oc apply -f cr-keycloak.yaml -n sso-dev
```

This command will retrieve the Grafana route and extract the host field from the returned JSON output. The host field contains the hostname for the Grafana route. In a short while, the Grafana instance should be accessible via this URL.

```
KEYCLOAK_URL=https://$(oc get route keycloak -n sso-dev -o jsonpath='{.spec.host}')
echo $KEYCLOAK_URL
```

The command below will store the user credentials for Keycloak in environment variables. To access the Keycloak Admin Console, you can then use the "echo" command to display the values of the environment variables.

```shell
KEYCLOAK_ADMIN=$(oc get secret credential-keycloak -o jsonpath='{.data.ADMIN_USERNAME}' -n sso-dev | base64 --decode)
KEYCLOAK_PASSWORD=$(oc get secret credential-keycloak -o jsonpath='{.data.ADMIN_PASSWORD}' -n sso-dev | base64 --decode)
```

```shell
echo $KEYCLOAK_URL
echo $KEYCLOAK_ADMIN
echo $KEYCLOAK_PASSWORD
```

## Populating Keycloak with Data Using the `kcadm.sh` Tool

To utilize the kcadm.sh command line tool, we must first log in. The following command will accomplish this.

```shell
oc exec keycloak-0 -ti -- /opt/eap/bin/kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user $KEYCLOAK_ADMIN --password $KEYCLOAK_PASSWORD
```

The following command will use the kcadm.sh tool to generate new users.

```shell
oc exec keycloak-0 -- /bin/bash -c "rm /tmp/keycloak-data-generator.sh && cat >/tmp/keycloak-data-generator.sh <<EOF

for i in {1..5}; do 
  /opt/eap/bin/kcadm.sh create users -s username=user-$i -s enabled=true -r test-realm 
  /opt/eap/bin/kcadm.sh set-password -r test-realm --username user-$i --new-password user-$i-password 
done

EOF
chmod u+x /tmp/keycloak-data-generator.sh && /tmp/keycloak-data-generator.sh
"
```

## Benchmark Keycloak with data


In order to run a load performance test using the `kcb.sh` tool, you will first need to download the tool and make sure it is accessible and executable.

https://www.keycloak.org/keycloak-benchmark/benchmark-guide/latest/configuration

```shell
./kcb.sh  --server-url="https://$(oc get route keycloak -o jsonpath=''{.spec.host}'' -n sso-dev)/auth" --realm-name=test-realm --users-per-sec=30 --measurement=300
```