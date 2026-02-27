---
name: k8s-simple-deploy
description: Assists with deploying simple apps to Kubernetes consisting of a deployment and a service.
---

# Kubernetes simple deploy skill

Use this skill when users want to deploy a basic app on the Kubernetes cluster.

## Instructions

- Expect the user to provide the name for an app they wish to deploy, along with a docker image reference.
  Optionally they may supply number of replicas and port number, but these are not required, and have defaults.
- Call the script `scripts/deploy-app.py` passing the supplied name, image, and optional parameters (# of replicas and port number) if supplied, in that order.
- The script generates a correct two-resource manifest (Deployment + Service) and writes it to the file `temp-manifest.yaml`.

You, the **agent**, are expected to apply the generated manifest `temp-manifest.yaml` against the current Kubernetes context.

## Example

```text
User: Deploy an app called "nginx" using image "docker/nginx" with 2 replicas on port 8080.
Agent: Invokes `scripts/deploy-app.py nginx docker/nginx 2 8080`
```

The "skill" creates a manifest named `temp-manifest.yaml` consisting of a Deployment + Service in one shot.
The Agent in turn applies the generated temp-manifest.yaml to the cluster.
