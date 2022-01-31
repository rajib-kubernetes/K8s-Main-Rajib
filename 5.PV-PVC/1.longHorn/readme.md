

```

helm show values longhorn/longhorn > /tmp/longhorn-values.yaml

helm install longhorn/longhorn --values /tmp/longhorn-values.yaml -n longhorn-storage --create-namespace


```


