# Java BuildConfig Example on OpenShift

This repository demonstrates how to build and deploy a simple Java application in **OpenShift** using a `BuildConfig` with source code from a Git repository.

---

## 📋 Prerequisites

* Access to an **OpenShift** cluster with the `oc` CLI installed and logged in.
* A GitHub account with a personal access token (PAT).
* Basic knowledge of OpenShift Projects, ImageStreams, and BuildConfigs.

---

## 🚀 Quick Start (Recommended)

If you want to use the existing code in this repository:

1. **Fork this repository** to your GitHub account.
2. Update the **GitHub secrets** and **BuildConfig** to point to your fork.
3. Apply the OpenShift configuration (steps below).

---

## 📦 Deployment on OpenShift

### 1️⃣ Create a New Project

```
oc new-project java-bc
```

### 2️⃣ Create Secrets for GitHub Access

```
oc create secret generic github-https \
    --type=kubernetes.io/basic-auth \
    --from-literal=username=GITHUB_USER \
    --from-literal=password=YOUR_PAT \
    --namespace=java-bc

oc create secret generic github-webhook \
    --from-literal=WebHookSecretKey=MY_WEBHOOK_SECRET \
    --namespace=java-bc
```

> Replace:
>
> * `GITHUB_USER` → your GitHub username
> * `YOUR_PAT` → your personal access token
> * `MY_WEBHOOK_SECRET` → your chosen webhook secret

### 3️⃣ Create an ImageStream

```
oc create imagestream java-app -n java-bc
```

### 4️⃣ Apply the BuildConfig

The repository already contains `buildconfig.yaml`. Update the Git URL if necessary:

```
source:
  git:
    uri: 'YOUR_FORK_REPO_URL'
    ref: main
```

Apply it:

```
oc apply -f buildconfig.yaml
```

### 5️⃣ Start the Build

```
oc start-build java-app-git --follow
```

---

## 🖥 Deploy the Application

Create a new app from the built image:

```bash
oc new-app java-bc/java-app:latest --name java-webapp
```

Expose it to the outside world:

```
oc expose service/java-webapp
```

Get the route:

```
oc get route java-webapp
```

You can now open the application in a browser or test with curl:

```
curl http://<route-host>
```

> Example response:
> `Hello World!`

---

## 🔍 Verification

```
oc get builds
oc get is java-app
oc get pods
oc get svc
oc get route java-webapp
```

---

## 📊 Workflow Diagram

```
flowchart LR
    A[GitHub Repository (Fork)] -->|Webhook / Manual Trigger| B(BuildConfig in OpenShift)
    B --> C[Source Build using JBoss WebServer + OpenJDK 11]
    C --> D[ImageStream java-app:latest]
    D --> E[Deployment / Pod]
```

This diagram shows:

1. Code is pushed to your GitHub fork.
2. Webhook or manual trigger starts a BuildConfig.
3. OpenShift builds the Java application image.
4. Image is stored in an ImageStream.
5. Image is used for Deployment or Pod creation.

---

## 📚 References

* [OpenShift BuildConfig Documentation](https://docs.openshift.com/container-platform/latest/cicd/builds/understanding-buildconfigs.html)
* [Maven Official Guide](https://maven.apache.org/guides/)

---

> **Note:** This program is intended to **test BuildConfig functionality**. It may not provide a fully functional web service for production.


