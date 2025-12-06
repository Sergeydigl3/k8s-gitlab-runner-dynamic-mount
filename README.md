# k8s-gitlab-runner-dynamic-mount

## Gen certs

```bash
export NS="gitlab-hooks"
export SVC="pvc-injector"
kubectl create ns $NS --dry-run=client -o yaml | kubectl apply -f -
openssl req -x509 -newkey rsa:2048 -nodes -keyout tls.key -out tls.crt -days 3650 -subj "/CN=${SVC}.${NS}.svc"   -addext "subjectAltName = DNS:${SVC}.${NS}.svc"
kubectl -n $NS create secret tls webhook-tls --cert=tls.crt --key=tls.key --dry-run=client -o yaml | kubectl apply -f -
export CA_BUNDLE=$(cat tls.crt | base64 | tr -d '\n')
cat CA_BUNDLE
```

Replace ca_bundle in hook. And run `k apply -f base-deploy/`

## Gitlab CI usage example
```yaml
stages:
  - input
  - output


input-job:
  stage: input
  image: alpine
  variables:
    SHARE_CLAIM_NAME: "test-disk-claim"
    MNT_PATH: "/mnt/data"
    SHARE_SUB_PATH: "project-a/builds"
  script:
    - echo "Runner works" > /mnt/data/test.log

output-job:
  stage: output
  image: alpine
  variables:
    SHARE_CLAIM_NAME: "test-disk-claim"
    MNT_PATH: "/mnt/data"
    SHARE_SUB_PATH: "project-a/builds"
  script:
    - cat /mnt/data/test.log
```

