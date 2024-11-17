# copyblocks-helm

## This repository is the beginning of a Helm chart for copyblocks, a tool that can be utilized to copy Mimir s3 data from bucket to bucket. 
Quite possibly it can be used to copy any blocks to any buckets. To date, I've only used with Mimir. 



## Pre-Reqs
You will need a secret containing your s3 creds. Do not store secrets in Git. Be mindful of extra characters if free forming the secret. use `echo -n secret | base64` Sample secret below:
```
apiVersion: v1
data:
  access_key_id: <>
  secret_access_key: <>
kind: Secret
metadata:
  name: copyblocks-s3-secret
  namespace: copyblocks
type: Opaque
```

## Clone and run locally
```
git clone https://github.com/pbmoses/copyblocks-helm.git
helm install copyblocks -f values.yaml -n copyblocks .
```

## Working as of today but improvements to come. Sample logs:
```
level=info ts=2024-11-17T22:16:53.278125776Z tenantID=__system__ block=01FCJ34990X7RH3ES0RDE9P2EV block_min_time=2021-08-07T00:00:00.481Z block_max_time=2021-08-08T00:00:00Z msg="copying block"
level=info ts=2024-11-17T22:16:53.304578002Z tenantID=__system__ block=01FCQ7QFFCXCKJX61T9B7NGVVE block_min_time=2021-08-09T00:00:00.48Z block_max_time=2021-08-10T00:00:00Z msg="copying block"
level=info ts=2024-11-17T22:16:53.309297503Z tenantID=__system__ block=01FCFGRXG7HDJ1E6G6QQZPJWGJ block_min_time=2021-08-06T00:00:00.479Z block_max_time=2021-08-07T00:00:00Z msg="copying block"
level=info ts=2024-11-17T22:16:53.316345485Z tenantID=__system__ block=01FCMNDRC4YHYMRCEH72KKE1CN block_min_time=2021-08-08T00:00:00.481Z block_max_time=2021-08-09T00:00:00Z msg="copying block"
```
