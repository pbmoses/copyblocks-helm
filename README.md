# copyblocks-helm

## This repository is the beginning of a Helm chart for copyblocks, a tool that can be utilized to copy Mimir s3 data from bucket to bucket. 
Quite possibly it can be used to copy any blocks to any buckets. To date, I've only used with Mimir. 

We do not need a serviceAccount (bout you could use one) or other objects, just a simple deployment with  the custom values. 


## Pre-Reqs
You will need a secret containing your s3 creds. Do not store secrets in Git. Be mindful of extra characters if creating the secret. Use `echo -n <secret string> | base64` to avoid extra chars. Sample secret below:
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

## Run natively from Helm 
```
helm repo add pbmoses-copyblocks https://pbmoses.github.io/copyblocks-helm/
helm install copyblocks pbmoses-copyblocks/copyblocks -f ./myvalues.yaml --dry-run --debug
```

## Clone and run locally
```
git clone https://github.com/pbmoses/copyblocks-helm.git
helm install copyblocks -f values.yaml -n copyblocks .
```

## Sample custom values for S3 back end 
```
# Default values for copyblocks.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: grafana/copyblocks 
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "r314-e189185"

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: false 
  # Automatically mount a ServiceAccount's API credentials?
  automount: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}
podLabels: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 8080

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi


autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

# Additional volumes on the output Deployment definition.
volumes: []
# - name: foo
#   secret:
#     secretName: mysecret
#     optional: false

# Additional volumeMounts on the output Deployment definition.
volumeMounts: []
# - name: foo
#   mountPath: "/etc/foo"
#   readOnly: true

nodeSelector: {}

tolerations: []

affinity: {}

env:
- name: S3_ACCESS_KEY
  valueFrom:
    secretKeyRef:
      key: access_key_id
      name: copyblocks-s3-secret 
- name: S3_SECRET_KEY
  valueFrom:
    secretKeyRef:
      key: secret_access_key
      name: copyblocks-s3-secret 

### your container arguments go here
### exampler per cloud provider https://pkg.go.dev/github.com/grafana/mimir/tools/copyblocks#section-readme

container_args:
  - -source.backend=s3
  - -copy-period=24h
  - -destination.backend=s3
  - -dry-run=false
  - -s3.source.bucket-name=cx-playground-shared-gem-blocks
  - -min-block-duration=13h
  - -s3.destination.access-key-id=$(S3_ACCESS_KEY)
  - -s3.destination.bucket-name=copyblocks-test-destination-pmoses
  - -s3.destination.endpoint=s3.amazonaws.com
  - -s3.destination.secret-access-key=$(S3_SECRET_KEY)
  - -s3.source.endpoint=s3.amazonaws.com
  - -s3.source.access-key-id=$(S3_ACCESS_KEY)
  - -s3.source.secret-access-key=$(S3_SECRET_KEY)
```

## Working as of today but improvements to come. Sample logs:
```
level=info ts=2024-11-17T22:16:53.278125776Z tenantID=__system__ block=01FCJ34990X7RH3ES0RDE9P2EV block_min_time=2021-08-07T00:00:00.481Z block_max_time=2021-08-08T00:00:00Z msg="copying block"
level=info ts=2024-11-17T22:16:53.304578002Z tenantID=__system__ block=01FCQ7QFFCXCKJX61T9B7NGVVE block_min_time=2021-08-09T00:00:00.48Z block_max_time=2021-08-10T00:00:00Z msg="copying block"
level=info ts=2024-11-17T22:16:53.309297503Z tenantID=__system__ block=01FCFGRXG7HDJ1E6G6QQZPJWGJ block_min_time=2021-08-06T00:00:00.479Z block_max_time=2021-08-07T00:00:00Z msg="copying block"
level=info ts=2024-11-17T22:16:53.316345485Z tenantID=__system__ block=01FCMNDRC4YHYMRCEH72KKE1CN block_min_time=2021-08-08T00:00:00.481Z block_max_time=2021-08-09T00:00:00Z msg="copying block"
```
