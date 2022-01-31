

```

helm show values longhorn/longhorn > /tmp/longhorn-values.yaml

helm install longhorn1 longhorn/longhorn --values /tmp/longhorn-values.yaml -n longhorn-system --create-namespace



```


