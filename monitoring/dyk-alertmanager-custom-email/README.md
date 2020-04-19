# DO YOU KNOW - How to use your own custom email template for notifications via Alertmanager

Contains an example on how you configure your own email template for Prometheus' Alertmanager in OpenShift.

## Add email template to the Alertmanager Secret

1. Open Alertmanager Secret:

```
$ kubectl -n openshift-monitoring edit secret alertmanager-main
``` 

2. Change secret and add the `myemailtemplate.tmpl` key and specify the content of your template as a base64 encoded string (find an example for a typical email template in the `myemailtemplate.tmpl` file in this repository).

```
apiVersion: v1
kind: Secret
metadata:
  name: alertmanager-main
  namespace: openshift-monitoring
type: Opaque
data:
  alertmanager.yaml: {BASE64_CONFIG}
  myemailtemplate.tmpl: {BASE64_TEMPLATE}
```

## Configure Alertmanager

1. Print the currently active Alertmanager configuration into file alertmanager.yaml:

```
$ kubectl -n openshift-monitoring get secret alertmanager-main --template='{{ index .data "alertmanager.yaml" }}' | base64 -d > alertmanager.yaml
```

2. Change the configuration in file alertmanager.yaml to your new configuration:

```
data:
 config.yaml: |
  global:
    resolve_timeout: 5m
  route:
    group_wait: 30s
    group_interval: 5m
    repeat_interval: 12h
    receiver: default
    routes:
    - match:
        alertname: Watchdog
      repeat_interval: 5m
      receiver: watchdog
  receivers:
  - name: default
  - name: watchdog
  - name: myemail
    email_configs:
    - send_resolved: true
      to: 'test@demo.com'
      html: '{{ template "email.mytemplate.html" . }}'
  templates:
  - 'myemailtemplate.tmpl'
  ```

  3. Apply the new configuration in the file:

  ```
  $ kubectl -n openshift-monitoring create secret generic alertmanager-main --from-file=alertmanager.yaml --dry-run -o=yaml |  oc -n openshift-monitoring replace secret --filename=-
  ```
