# conjur-kubernetes-authenticator

On DockerHub: https://hub.docker.com/r/cyberark/conjur-kubernetes-authenticator/

## What's inside ?

The sidecar is designed to have a light footprint both in terms of storage and memory consumption. It has very few components

+ a static binary for the authenticator
+ the sleep binary from busybox for debugging
+ the tar binary from busybox to meet the requirement of the authentication service

## Configuration

The sidecar authenticator is configured entirely through environment variables. These are listed below.

## Orchestrator
The values in this section can be retrieved using the [downwards API](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information)
- `MY_POD_NAME`: Pod name
- `MY_POD_NAMESPACE`: Pod namespace

## Conjur
- `CONJUR_ACCOUNT`: Conjur account name (v4)
- `CONJUR_AUTHN_URL`: URL pointing to authenticator service endpoint
- `CONJUR_AUTHN_LOGIN`: Host login for pod e.g. `namespace/service_account/some_service_account`
- `CONJUR_SSL_CERTIFICATE`: Public SSL cert for Conjur connection (v4)

Flow:

The sidecar's process logs its flow to `stdout` and `stderr`. 
+ Exponential backoff is exercised when an error occurs
+ Sidecar will re-login when certificate has expired

1. Sidecar goes through login by presenting certificate signing request (CSR) -> Server (authn-k8s running inside the appliance) injects signed client certificate out of band into requesting pod
2. Sidecar picks up signed client certificate, deletes it from disk and uses to authenticator via mutual TLS -> Server responds with auth token (retrieved via authn-local) encrypted with the public key of the sidecar.
3. Sidecar decrypts the auth token and writes it to to the shared memory volume (`/run/conjur/access-token`) 
4. Sidecar proceeds to authenticate time and time again
