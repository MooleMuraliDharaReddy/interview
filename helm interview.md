# **Helm Part 1**  

## **1. What is Helm?**  
Helm is a **package manager for Kubernetes** that simplifies application deployment and management. Think of it like **APT for Ubuntu** or **Yum for CentOS**, but for Kubernetes.

### **Key Features of Helm:**  
- **Charts** → Prepackaged Kubernetes applications (like Docker images for containers).  
- **Templating Engine** → Uses **Go templates** to customize Kubernetes YAML files.  
- **Release Management** → Track, update, and roll back application versions.  
- **Dependency Management** → Define and install related services (e.g., app with Redis).  

### **Why use Helm?**  
- **Simplifies Deployment** → Instead of managing multiple Kubernetes YAML files, Helm allows you to define everything in a single chart.  
- **Versioning & Rollbacks** → Easily update or revert to previous versions of your app.  
- **Configuration Management** → Override settings using values files or command-line arguments.

---

## **2. What is the key difference between Helm 2 and Helm 3?**  
| Feature          | Helm 2 | Helm 3 |
|-----------------|--------|--------|
| **Tiller (Server-Side Component)** | Uses Tiller, requires RBAC setup | Removed for better security |
| **Security** | Tiller had full cluster-wide access | Uses Kubernetes RBAC, safer |
| **Release Namespaces** | All releases stored cluster-wide | Releases are namespace-scoped |
| **CRDs (Custom Resource Definitions)** | Managed separately | Managed within Helm |
| **Chart Dependencies** | Managed via `requirements.yaml` | Uses `Chart.yaml` |

**Why Helm 3?**  
- No need to manage **Tiller**, reducing security risks.  
- Simplifies **RBAC** and permissions.  
- Better **upgrade and rollback** experience.

---

## **3. What is the purpose of Helm charts in Kubernetes?**  
A **Helm chart** is a collection of YAML files and templates that define a Kubernetes application.  

### **Key Components of a Helm Chart**  
- `Chart.yaml` → Metadata about the chart (name, version, dependencies).  
- `values.yaml` → Default configuration values (can be overridden).  
- `templates/` → Templated Kubernetes manifests (Deployment, Service, etc.).  
- `charts/` → Dependencies (other Helm charts needed for the app).  

### **Example: Helm Chart Structure**  
```
my-app/
│── Chart.yaml        # Metadata about the chart
│── values.yaml       # Default values for the chart
│── templates/        # Kubernetes YAML templates
│   ├── deployment.yaml
│   ├── service.yaml
│── charts/           # Chart dependencies
```

---

## **4. How do you install a Helm chart into a Kubernetes cluster?**  
### **Steps to Install a Helm Chart**  
1. **Add a Helm repository (if not already added):**  
   ```sh
   helm repo add stable https://charts.helm.sh/stable
   ```
2. **Update Helm repositories:**  
   ```sh
   helm repo update
   ```
3. **Install the chart:**  
   ```sh
   helm install my-app stable/nginx --namespace default
   ```

### **Explanation of the Command**  
- `helm install` → Installs a new Helm release.  
- `my-app` → Name of the release.  
- `stable/nginx` → Chart to install from the repository.  
- `--namespace default` → Deploys in the default namespace.

---

## **5. What commands do we use in real-time for Helm? Can you explain in detail?**  
Here are the most commonly used Helm commands in production:

### **Installing a Helm Chart**
```sh
helm install my-app stable/nginx --values custom-values.yaml
```

### **Listing Installed Releases**
```sh
helm list
```
- Shows deployed Helm releases.

### **Upgrading an Existing Release**
```sh
helm upgrade my-app stable/nginx --values new-values.yaml
```

### **Rolling Back to a Previous Version**
```sh
helm rollback my-app 1
```

### **Viewing the History of a Release**
```sh
helm history my-app
```

### **Uninstalling a Helm Release**
```sh
helm uninstall my-app
```

---

## **6. How do you upgrade a Helm chart?**  
If you want to apply changes to a deployed Helm release, use:
```sh
helm upgrade my-app stable/nginx --values updated-values.yaml
```
**Best Practices for Helm Upgrades**  
- Always **backup** your current values using:  
  ```sh
  helm get values my-app > backup-values.yaml
  ```
- Test upgrades using `helm diff upgrade` (requires **helm diff plugin**).
- If something goes wrong, use `helm rollback`.

---

## **7. How can you roll back a Helm chart to a previous version?**  
Each Helm release maintains a revision history. To roll back:
```sh
helm rollback my-app 1
```
To check available revisions:
```sh
helm history my-app
```

---

## **8. How do you explore Helm releases using the `helm get` command?**  
Retrieve installed release details:
```sh
helm get all my-app
```
- `helm get values my-app` → Shows current values.  
- `helm get manifest my-app` → Shows Kubernetes resources.

---

## **9. How can you discover chart details with the `helm show` command?**  
To view details about a chart before installing it:
```sh
helm show chart stable/nginx
```
To check default values:
```sh
helm show values stable/nginx
```

---

## **10. How do you inspect the Helm environment? Do you use any command?**  
```sh
helm env
```
Shows paths and environment variables used by Helm.

---

## **11. How do you ensure the quality of a Helm chart?**  
- **Lint the chart** for errors:  
  ```sh
  helm lint my-chart/
  ```
- **Use best practices:**  
  - Avoid hardcoding values in templates.  
  - Use `helm template` for testing before deploying.  
  ```sh
  helm template my-chart/
  ```
- **Use Helm chart testing tools** like `helm unittest`.

---

## **12. How do you use Kubernetes YAML files in Helm chart YAML format?**  
Helm templates allow dynamic values using Go templating.

Example: **Deployment in Kubernetes**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
```
**Same Deployment in Helm Template**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicas }}
```

---

## **13. How do you pass environment variables into a Helm chart during installation or upgrade?**  
Use `--set` flag:
```sh
helm install my-app stable/nginx --set replicaCount=3
```
Or define in `values.yaml` and use:
```sh
helm install my-app -f values.yaml stable/nginx
```

---

## **14. How can you manage multiple values files within a single Helm chart?**  
You can use multiple values files:
```sh
helm install my-app -f values1.yaml -f values2.yaml stable/nginx
```
Later, during upgrade:
```sh
helm upgrade my-app -f values3.yaml stable/nginx
```

---

# **Helm Part 2**  

## **15. What are Helm repositories, and how do you add a repository in Helm?**  

### **What is a Helm Repository?**  
A **Helm repository** is a location where Helm charts are stored and can be fetched from. Some popular public repositories are:  
- **Helm Stable Repository**: `https://charts.helm.sh/stable` (deprecated, but widely used)  
- **Bitnami Repository**: `https://charts.bitnami.com/bitnami`  
- **Kubernetes Community Charts**: `https://charts.k8s.io/`  

### **Adding a Repository**  
To add a Helm repository:  
```sh
helm repo add my-repo https://charts.example.com/
```

### **Updating Repositories**  
To ensure you have the latest charts:  
```sh
helm repo update
```

### **Listing Available Repositories**  
```sh
helm repo list
```

### **Searching for Charts in a Repository**  
```sh
helm search repo my-repo/nginx
```

---

## **16. How do you manage dependencies within a Helm chart?**  

### **Why Do We Need Dependencies?**  
If an application depends on **Redis, MySQL, or any other service**, we can manage these dependencies within the Helm chart.  

### **Defining Dependencies in Chart.yaml**  
Add dependencies inside `Chart.yaml` under `dependencies:`  

```yaml
dependencies:
  - name: redis
    version: "17.3.2"
    repository: "https://charts.bitnami.com/bitnami"
```

### **Updating Dependencies**  
After defining dependencies, run:  
```sh
helm dependency update
```
This will download dependencies and store them inside the `charts/` directory.

### **Installing Dependencies Alongside Your App**  
```sh
helm install my-app ./my-chart
```

---

## **17. How can you secure Helm charts and ensure sensitive data is not exposed?**  

### **Best Practices for Helm Security**  
1. **Use Kubernetes Secrets Instead of Hardcoded Values**  
   Instead of storing credentials in `values.yaml`, store them as **Kubernetes Secrets**.  
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: db-secret
   type: Opaque
   data:
     password: {{ .Values.dbPassword | b64enc }}
   ```
   
2. **Use Helm Secrets Plugin**  
   Encrypts values using **Helm Secrets** plugin (`helm-secrets`).
   ```sh
   helm secrets enc values.yaml
   ```

3. **Restrict Access to Helm Repositories**  
   - Use **private repositories** for sensitive applications.  
   - Authenticate with credentials when pulling charts.  

---

## **18. How can you integrate Helm into a CI/CD pipeline for Kubernetes deployments?**  

### **CI/CD Workflow for Helm**  
1. **Lint Helm Charts**:  
   ```sh
   helm lint my-chart/
   ```
2. **Deploy to a Kubernetes Cluster (Example in GitHub Actions)**  
   ```yaml
   - name: Deploy with Helm
     run: helm upgrade --install my-app ./my-chart --namespace production
   ```
3. **Rollback if Needed**  
   ```sh
   helm rollback my-app 1
   ```

### **Helm in Jenkins Pipeline Example**  
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy with Helm') {
            steps {
                sh 'helm upgrade --install my-app ./my-chart --namespace prod'
            }
        }
    }
}
```

---

## **19. How do you customize Helm charts to fit your application's requirements?**  

### **1. Override Values During Installation**  
```sh
helm install my-app ./my-chart --set image.tag=2.0.0
```

### **2. Use a Custom `values.yaml` File**  
Modify `values.yaml` for different environments:
```yaml
replicaCount: 3
image:
  repository: my-app
  tag: "2.0.0"
```
Then install:
```sh
helm install my-app -f values.yaml ./my-chart
```

---

## **20. How can you create your own Helm chart from scratch?**  

### **Steps to Create a Helm Chart**  
1. **Generate a New Chart**  
   ```sh
   helm create my-chart
   ```
   This creates a directory structure:
   ```
   my-chart/
   ├── Chart.yaml
   ├── values.yaml
   ├── templates/
   ```

2. **Modify `values.yaml`**  
   ```yaml
   replicaCount: 3
   image:
     repository: nginx
     tag: latest
   ```

3. **Deploy the Chart**  
   ```sh
   helm install my-app ./my-chart
   ```

---

## **21. What are common issues that occur when using Helm?**  

### **1. Helm Chart Fails to Deploy**  
- **Check logs**  
  ```sh
  kubectl logs -l app=my-app
  ```
- **Check status**  
  ```sh
  helm status my-app
  ```

### **2. Helm Release Not Found**  
- Ensure the release exists:
  ```sh
  helm list --all-namespaces
  ```

### **3. Error: "chart dependencies not updated"**  
- Run:  
  ```sh
  helm dependency update
  ```

---

## **22. How do you migrate from Helm 2 to Helm 3?**  

### **1. Install Helm 3 Plugin for Migration**  
```sh
helm plugin install https://github.com/helm/helm-2to3
```

### **2. Convert Helm 2 Releases to Helm 3**  
```sh
helm 2to3 convert my-release
```

### **3. Verify the Migration**  
```sh
helm list --all-namespaces
```
