# Lab 2 — hello-app on Kubernetes

Your copy of this repository is your submission. The lab sheet is on the course site; this file is where you record what you did.

What's here:

| File | What it is |
|---|---|
| `src/`, `index.html` | a small React page that shows which Pod answered |
| `nginx.conf` | serves the page, and answers `/whoami` with the Pod name |
| `Dockerfile` | builds the page, then serves it with nginx on port 80 |

You don't change any of these. You add `hello-app.yaml` and fill in the sections below.

## Before you push, check

- [ ] `hello-app.yaml` is in the root of this repository
- [ ] `screenshot.png` is in the root of this repository
- [ ] every section below is filled in

---

## 1. My image

Docker Hub image I deployed (for example `stark1802/hello-app:v1`):

```
Contains	Visibility	Scout
stark1802/hello-app
about 1 hour ago
image
Public	
Inactive
```

## 2. Pods across nodes

Output of `kubectl get pods -o wide` after you applied `hello-app.yaml`:

```C:\Users\shiva keerthi mp\Downloads\Lab_Assignment_Devops\ms-cloud-devops-lab2>kubectl apply -f hello-app.yaml
Warning: resource deployments/hello-app is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/hello-app configured
Warning: Detected changes to resource hello-app which is currently being deleted.
service/hello-app unchanged

C:\Users\shiva keerthi mp\Downloads\Lab_Assignment_Devops\ms-cloud-devops-lab2>kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
hello-app-744fcb7bc5-ckw66   1/1     Running   0          53m
hello-app-744fcb7bc5-j8kjv   1/1     Running   0          53m
hello-app-744fcb7bc5-pqnlk   1/1     Running   0          31m
hello-app-744fcb7bc5-skjmw   1/1     Running   0          53m
```

## 3. The Service and its endpoints

Output of `kubectl get service hello-app` and `kubectl get endpointslices -l kubernetes.io/service-name=hello-app`:

```
C:\Users\shiva keerthi mp\Downloads\Lab_Assignment_Devops\ms-cloud-devops-lab2>kubectl get svc hello-app
NAME        TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hello-app   LoadBalancer   10.96.46.196   <pending>     8000:31097/TCP   63m


C:\Users\shiva keerthi mp\Downloads\Lab_Assignment_Devops\ms-cloud-devops-lab2>kubectl get endpointslices -l kubernetes.io/service-name=hello-app
NAME              ADDRESSTYPE   PORTS   ENDPOINTS                             AGE
hello-app-z4krr   IPv4          80      10.244.1.12,10.244.2.13,10.244.2.14   63m
```

## 4. Browser

Add `screenshot.png`: the page at `http://localhost:8000` with at least two Pods in the "Answers so far" table.

## 5. The four traps

For each trap, write what you saw and why it happened, in two or three lines of your own words.

**Trap 1 — the local image.**
The Pods initially showed ImagePullBackOff because the Docker image was available only on my laptop. Kubernetes worker nodes could not access the local image. I pushed the image to Docker Hub and recreated the Deployment using the public image.

**Trap 2 — the selector typo.**
When I changed the Service selector from app: hello-app to app: helo-app, the EndpointSlice became empty. This happened because the Service could not find Pods matching the incorrect label. After correcting the selector, the application started working again.

**Trap 3 — the wrong targetPort.**
When I changed targetPort: 80 to targetPort: 8000, the EndpointSlice still listed the Pods but showed port 8000. The application stopped responding because nginx listens on port 80. Changing the targetPort back to 80 fixed the issue.
**Trap 4 — the EXTERNAL-IP address.**

\The LoadBalancer Service showed an 10.96.46.196 address that was pending or inaccessible from my browser because Docker Desktop uses a private network. I accessed the application through http://localhost:8000 instead, using port forwarding.
## 6. Your YAML vs the generated YAML
Two fields present in the generated YAML but not in my file were:
1. creationTimestamp: Records when the Kubernetes resource was created.
2. strategy: Defines how Kubernetes updates the Deployment when a new version is released.
My YAML works without these fields because Kubernetes automatically assigns default values and manages resource metadata. Only the required configuration fields need to be specified in my manifest.

Name two fields in the `--dry-run=client -o yaml` output that were not in your file. Why does your file still work without them?
