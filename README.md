# MESS: Most Excellent Secret Scriber

MESS reads content from secrets and writes it to your node
filesystems.

## Example

You would like to write secret credentials to the file `/etc/flagfile`
on your workers. The desired file content is stored in the `flagfile`
secret, under the key `flagfile`:

```
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
  namespace: default
type: Opaque
stringData:
  flagfile: |
    This is a secret.
```

You would use the following `MessConfig` resource:

```
apiVersion: v1
kind: MessConfig
metadata:
  name: write-flag-file
type: Opaque
spec:
  nodeSelector:
    node-role.kubernetes.io/worker: ""
  files:
    - path: /etc/flagfile
      mode: '0644'
      secretKeyRef:
        name: example-secret
        key: flagfile
```

## Configuration

`updatePolicy` can be one of:

- `always` -- always write the file. You probably don't want to use
  this.

- `ifNewer` -- only write the file if the modification time of the
  secret is more recent than the modification time of the target file.

- `ifChanged` -- only write the file if the hash of the secret is different
  from the hash of the target file. NEWS will *always* check the
  content hash when you use this policy.

- `ifNewerAndChanged` -- First check the modification of the secret
  vs. the file. If the secret is more recent, check the hash of the
  file and compare it to the secret. Only write the file if both
  checks pass.

  This is the default policy.
