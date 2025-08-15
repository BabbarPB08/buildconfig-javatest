Alright — here’s the **final, fully explained `README.md`** with:

* Only **fork-and-use** instructions (no "start from scratch")
* **Pipeline section** included
* **Fixed Mermaid diagram** (GitHub-compatible)
* **Webhook setup steps** so BuildConfig auto-builds on GitHub commits
* Clear S2I explanation

---

````markdown
# Java BuildConfig + Deployment Example on OpenShift

This repository demonstrates how to **fork**, build, and deploy a Java application on **OpenShift** using a `BuildConfig` with **Source-to-Image (S2I)** and a **GitHub webhook** for automatic rebuilds.

---

## 📋 Prerequisites

- Access to an **OpenShift** cluster with the `oc` CLI installed and logged in.
- A **GitHub** account and the ability to fork this repository.
- Basic knowledge of OpenShift Projects, ImageStreams, BuildConfigs, and Deployments.

---

## 🚀 Quick Start

1. **Fork this repository** into your own GitHub account.

2. **Create a project in OpenShift**:
   ```bash
   oc new-project java-bc
````

3. **Create secrets for GitHub access**:

   ```bash
   oc create secret generic github-https \
       --type=kubernetes.io/basic-auth \
       --from-literal=username=GITHUB_USER \
       --from-literal=password=YOUR_PAT \
       --namespace=java-bc
   ```

   ```bash
   oc create secret generic github-webhook \
       --from-literal=WebHookSecretKey=MY_WEBHOOK_SECRET \
       --namespace=java-bc
   ```

   Replace:

   * `GITHUB_USER` → your GitHub username
   * `YOUR_PAT` → your GitHub personal access token
   * `MY_WEBHOOK_SECRET` → your chosen webhook secret

4. **Create an ImageStream**:

   ```bash
   oc create imagestream java-app -n java-bc
   ```

5. **Apply the BuildConfig**:
   Save as `buildconfig.yaml`:

   ```yaml
   apiVersion: build.openshift.io/v1
   kind: BuildConfig
   metadata:
     name: java-app-git
     namespace: java-bc
   spec:
     output:
       to:
         kind: ImageStreamTag
         name: 'java-app:latest'
     resources: {}
     successfulBuildsHistoryLimit: 2
     failedBuildsHistoryLimit: 1
     strategy:
       type: Source
       sourceStrategy:
         from:
           kind: ImageStreamTag
           namespace: openshift
           name: 'jboss-webserver57-openjdk11-tomcat9-openshift-ubi8:latest'
         incremental: false
     source:
       type: Git
       git:
         uri: 'https://github.com/<your-user>/<your-fork>.git'
         ref: main
       sourceSecret:
         name: github-https
     triggers:
       - type: ConfigChange
       - type: ImageChange
         imageChange: {}
       - type: GitHub
         github:
           secretReference:
             name: github-webhook
     runPolicy: Serial
   ```

   Apply it:

   ```bash
   oc apply -f buildconfig.yaml
   ```

---

## 🔔 Set Up GitHub Webhook for Auto-Builds

OpenShift supports **S2I builds** triggered by GitHub webhooks.

1. **Get the webhook URL**:

   ```bash
   oc describe bc java-app-git | grep webhook
   ```

   Example:

   ```
   https://api.cluster.example.com:6443/apis/build.openshift.io/v1/namespaces/java-bc/buildconfigs/java-app-git/webhooks/<secret>/github
   ```

2. **Add webhook in GitHub**:

   * Go to your forked repo → **Settings** → **Webhooks** → **Add webhook**
   * Payload URL: paste the URL above
   * Content type: `application/json`
   * Secret: use the value from `MY_WEBHOOK_SECRET`
   * Select **"Just the push event"**
   * Save

3. **Test**: Commit to `main` → OpenShift auto-starts a build:

   ```bash
   oc get builds
   ```

---

## 🖥 Deploy the Application

Once your image is built:

```bash
oc new-app java-bc/java-app:latest --name java-webapp
oc expose service/java-webapp
```

Check the route:

```bash
oc get route java-webapp
```

Test in browser or curl:

```bash
curl http://<your-route>
```

You should see:

```
Hello World!
```

---

## 📊 Build & Deploy Pipeline

```mermaid
flowchart TD
    A[GitHub Fork] -->|Push Commit| B[GitHub Webhook]
    B --> C[OpenShift BuildConfig (S2I)]
    C --> D[ImageStream java-app:latest]
    D --> E[Deployment / Pod]
    E --> F[Service]
    F --> G[Route Exposed to Public]
```

---

## 📚 Notes

* **S2I (Source-to-Image)**: This process takes your source code, builds it with a builder image (JBoss WebServer + OpenJDK 11), and produces a runnable image.
* BuildConfig triggers:

  * **GitHub Webhook** → build on code push.
  * **ImageChange** → redeploy on new image.
  * **ConfigChange** → rebuild if config changes.
* This example is for **testing BuildConfig + S2I** only — not optimized for production.

---

## 🔗 References

* [OpenShift BuildConfig Documentation](https://docs.openshift.com/container-platform/latest/cicd/builds/understanding-buildconfigs.html)
* [S2I Documentation](https://docs.openshift.com/container-platform/latest/cicd/builds/understanding-image-builds.html)
* [Maven Official Guide](https://maven.apache.org/guides/)

```