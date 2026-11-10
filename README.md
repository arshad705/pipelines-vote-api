# tekton-pipelines-demo

A small voting API written in Go with [Gin](https://github.com/gin-gonic/gin). It is the backend used in the OpenShift Pipelines (Tekton) demo. Votes are kept in memory, so they are lost when the server restarts.

The server listens on port `9000`.

## Endpoints

| Method | Path          | Description                                         |
| ------ | ------------- | --------------------------------------------------- |
| GET    | `/healthz`    | Health check. Returns `{"status": "ok"}`.           |
| GET    | `/vote`       | Current vote counts, e.g. `{"a": 3, "b": 1}`.       |
| GET    | `/vote/total` | Number of voters who have voted, e.g. `{"total": 4}`. |
| POST   | `/vote`       | Cast or change a vote.                              |

### Casting a vote

```sh
curl -X POST http://localhost:9000/vote \
  -d '{"voter_id": "user-1", "vote": "a"}'
```

`vote` should be `"a"` or `"b"`. Each `voter_id` holds one vote: posting again with the same `voter_id` replaces that voter's earlier vote.

## Running locally

Requires Go. Dependencies are vendored.

```sh
go run -mod=vendor .
```

Then check it is up:

```sh
curl http://localhost:9000/healthz
```

## Building the container image

The `Dockerfile` uses a multi-stage build and produces a minimal `scratch` image. Its builder image comes from the OpenShift internal registry (`image-registry.openshift-image-registry.svc:5000/openshift/golang`), so build it inside an OpenShift cluster, or swap in a public Go image such as `golang:1.22` to build elsewhere.

## Deploying to Kubernetes / OpenShift

Manifests are in [`k8s/`](k8s/):

```sh
kubectl apply -f k8s/
```

This creates the `pipelines-vote-api` Deployment and a ClusterIP Service on port `9000`.
