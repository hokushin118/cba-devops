# DevOps Configuration

This repository provides the infrastructure and configuration for the CBA
project's cloud-native microservices.

## Environment Profiles

This project utilizes distinct environment profiles to manage infrastructure
and application configurations for different stages of development and
deployment.

**Available Profiles:**

* **development (default):**
    * Used for local development and testing.
    * Provides a development-friendly configuration.
* **docker:**
    * Used for running microservices within Docker containers.
    * Configures the application for a containerized environment.
* **production:**
    * Used for deploying microservices in a production environment.
    * Optimized for performance and security.

**Profile-Specific Environment Variables:**

Environment variables specific to each profile are defined in `.env.<profile>`
files located in the project's root directory.

* `.env`: Default environment variables (development profile).
* `.env.docker`: Environment variables for the Docker profile.
* `.env.production`: Environment variables for the production profile.

**Important Security Note (Production):**

* The `.env.production` file may contain sensitive information (e.g., database
  credentials, API keys). **Never commit this file to a version control
  repository.** Use secure methods for deploying production secrets, such as
  environment variables managed by your deployment platform or dedicated secret
  management tools.

**Setting the Active Profile:**

The active profile is determined by the `APP_SETTINGS` environment variable.

* Example (setting the Docker profile):
    ```bash
    export APP_SETTINGS=docker
    ```

## Local Development

To run the microservices locally for development and testing, follow these
steps:

1. **Clone the Repository:**
    * Clone the `cba-devops` repository to your local machine.
   ```bash
   git clone [https://github.com/hokushin118/cba-devops.git](https://github.com/hokushin118/cba-devops.git)
   ```

2. **Navigate to the Project Directory:**
    * Change your current directory to the cloned repository.
   ```bash
   cd cba-devops
   ```

3. **Start Infrastructure Services with Docker Compose:**
    * Use Docker Compose to launch the necessary infrastructure services (e.g.,
      databases, message queues, Keycloak).
    * This command starts the services defined in both `docker-compose.yml`
      and `docker-compose.<profile>.yml`.
   ```bash
   docker compose -f docker-compose.yml -f docker-compose.<profile>.yml up --build
   ```
    * Example (test profile):
   ```bash
   docker compose -f docker-compose.yml -f docker-compose.test.yml up --build
   ```
    * The `--build` flag will build any images that are not already built.

4. **Run the Microservices:**
    * Once the infrastructure services are running, you can start the
      individual microservices. Refer to the specific microservice's
      documentation for instructions on how to run it locally (e.g.,
      using `python app.py` or similar).

**Important Notes:**

* Ensure Docker and Docker Compose are installed on your machine.
* `docker-compose.<profile>.yml` (e.g., `docker-compose.test.yml`) includes
  services required for the specified profile, such as test databases or mock
  services.
* Refer to the individual microservice's README for specific setup instructions
  and dependencies.

**Stopping Services:**

* To stop the services, press **Ctrl+C** in the terminal where they are
  running, or execute:
    ```bash
    docker compose -f docker-compose.yml -f docker-compose.<profile>.yml down
    ```
* Example (test profile):
    ```bash
    docker compose -f docker-compose.yml -f docker-compose.test.yml down
    ```
* To remove volumes as well, add the `-v` flag:
    ```bash
    docker compose -f docker-compose.yml -f docker-compose.test.yml down -v
    ```

**Extending Docker Compose Configuration:**

* For details on extending Docker Compose configurations,
  see: [Extend your Compose files](https://docs.docker.com/compose/how-tos/multiple-compose-files/extends)

## Environment Variables

Environment variables for each profile are configured in the `.env` files.

## Deployment on Kubernetes

This guide outlines the steps to deploy microservices on
a [Kubernetes](https://kubernetes.io) cluster.

**Configuration Files:**

* [Kubernetes](https://kubernetes.io) configuration files are located in
  the `k8s/` directory.

**Deployment Steps:**

To deploy a microservice to [Kubernetes](https://kubernetes.io), use the
following commands:

1. **Namespace**: Apply the namespace yaml first. All other resources will be
   created within this namespace (**cba-dev**).

```bash
kubectl apply -f .k8s/namespaces/cba-dev-ns.yml
```

2. **ConfigMap**: Apply the microservice ConfigMap yaml next. The Deployment
   depends on it.

```bash
kubectl apply -n cba-dev -f .k8s/configmaps/<name of microservice>-srv-cm.yml
```

For example:

```bash
kubectl apply -n cba-dev -f .k8s/configmaps/account-srv-cm.yml
```

3. **Secret**: Apply the microservice Secret yaml. The Deployment also depends
   on it.

```bash
kubectl apply -n cba-dev -f .k8s/secrets/<name of microservice>-srv-secret.yml
```

For example:

```bash
kubectl apply -n cba-dev -f .k8s/secrets/account-srv-secret.yml
```

4. **Deployment**: Apply the microservice Deployment yaml last. It depends on
   the Namespace, ConfigMap, and Secret.

```bash
kubectl apply -n cba-dev -f .k8s/deployments/<name of microservice>-srv-deployment.yml
```

For example:

```bash
kubectl apply -n cba-dev -f .k8s/deployments/account-srv-deployment.yml
```

5. **Service**: Apply the microservice Service yaml. It depends on the Pods
   created by the Deployment.

```bash
kubectl apply -n cba-dev -f .k8s/services/<name of microservice>-srv-service.yml
```

For example:

```bash
kubectl apply -n cba-dev -f .k8s/services/account-srv-service.yml
```

6. **HPA**: Apply the microservice HPA yaml. It depends on the Deployment.

```bash
kubectl apply -n cba-dev -f .k8s/hpas/<name of microservice>-srv-hpa.yml
```

For example:

```bash
kubectl apply -n cba-dev -f .k8s/hpas/account-srv-hpa.yml
```

**Verification:**

Check the pods are running with:

```bash
kubectl get pods -n cba-dev
```

Check the Service is created with:

```bash
kubectl get services -n cba-dev
```

Check the HPA is created with:

```bash
kubectl get hpa -n cba-dev
```

## Deployment on Red Hat OpenShift with Tekton

This guide outlines the steps to deploy your application
on [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift)
using
[Tekton](https://tekton.dev) Pipelines.

[Tekton](https://tekton.dev) is a cloud-native solution for building CI/CD
systems. It consists of [Tekton](https://tekton.dev) Pipelines, which provides
the building blocks, and of supporting components, such as Tekton CLI and
Tekton Catalog, that make Tekton a complete ecosystem. For more information,
see the [Tekton documentation](https://tekton.dev/docs/).

**Prerequisites:**

* **Red Hat OpenShift Cluster:** You need access to
  a [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift)
  cluster.
* **Red Hat OpenShift CLI (oc):** Install the OpenShift CLI. Download it
  from [Red Hat Downloads](https://access.redhat.com/downloads/content/290/ver=4.17/rhel---9/4.17.16/x86_64/product-software).
    * Verify installation: `oc version`
* **Tekton CLI (tkn):** Install the Tekton CLI. Download it
  from [Tekton CLI Releases](https://github.com/tektoncd/cli/releases).
    * Example (RHEL 9):
        ```bash
        rpm -Uvh [https://github.com/tektoncd/cli/releases/download/v0.37.1/tektoncd-cli-0.37.1_Linux-64bit.rpm](https://github.com/tektoncd/cli/releases/download/v0.37.1/tektoncd-cli-0.37.1_Linux-64bit.rpm)
        ```
      or
        ```bash
        curl -LO [https://github.com/tektoncd/cli/releases/download/v0.37.1/tkn_0.37.1_Linux_x86_64.tar.gz](https://github.com/tektoncd/cli/releases/download/v0.37.1/tkn_0.37.1_Linux_x86_64.tar.gz)
        sudo tar xvzf tkn_0.37.1_Linux_x86_64.tar.gz -C /usr/local/bin/ tkn
        ```
    * Verify installation: `tkn version`
* **Tekton Pipelines:** Tekton Pipelines must be installed on your [Red Hat
  OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift)
  cluster.

**Configuration Files:**

* [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift)
  configuration files: `openshift/` directory.
* Tekton custom Task files: `openshift/tekton/tasks/` directory.
* Tekton Pipeline files: `openshift/tekton/pipelines/` directory.

**Setup Steps:**

1. Login to the [Red Hat OpenShift](https://www.redhat.
   com/en/technologies/cloud-computing/openshift) cluster, using the following
   command:

```bash
oc login --token=<token> --server=https://api.,,.p1.openshiftapps.com:6443
```

2. Install [Tekton Pipeline](https://github.com/tektoncd/pipeline/releases)
   on [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift)
   using the following command:

```bash
oc apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.68.0/release.yaml
```

3. [Download](https://github.com/tektoncd/cli/releases) and install [Tekton
   CLI](https://tekton.dev/docs/cli) on your machine. For example, to download
   and install **Tekton CLI** on **RHEL 9**, use the following commands:

```bash
rpm -Uvh https://github.com/tektoncd/cli/releases/download/v0.37.1/tektoncd-cli-0.37.1_Linux-64bit.rpm
```

or

```bash
curl -LO https://github.com/tektoncd/cli/releases/download/v0.37.1/tkn_0.37.1_Linux_x86_64.tar.gz
sudo tar xvzf tkn_0.37.1_Linux_x86_64.tar.gz -C /usr/local/bin/ tkn
```

After installation (using either method), verify that Tekton is installed
correctly, using the following command:

```bash
tkn version
```

4. To create a workspace for pipeline
   on [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift),
   use the following
   commands:

```bash
oc create -f .openshift/cba-pipeline-pvc.yml
```

5. Apply the Secret yaml.

```bash
oc apply -f .openshift/cba-pipeline-secret.yml
```

6. To create custom [Tekton](https://tekton.dev) tasks for pipeline
   on [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift),
   use the following commands:

```bash
oc apply -f .openshift/tekton/tasks/run-cleanup-workspace.yml 
oc apply -f .openshift/tekton/tasks/run-flake8-lint.yml 
oc apply -f .openshift/tekton/tasks/run-nose-tests.yml 
oc apply -f .openshift/tekton/tasks/run-trivy-scan.yml 
oc apply -f .openshift/tekton/tasks/run-database-migration.yml 
oc apply -f .openshift/tekton/tasks/run-revert-database-migration.yml 
```

Apply the run-github-clone-w-token.yml if you are using a private repository.

```bash
oc apply -f .openshift/tekton/tasks/run-github-clone-w-token.yml 
```

To verify the created custom tasks, using the following command:

```bash
oc get tasks
```

7. The **clone** task requires the **git-clone**, the **build** task
   requires **buildah** and the **deploy** task requires the
   ""openshift-client"" tasks from the **Tekton Hub**, use the following
   commands to install them:

```bash
tkn hub install task git-clone
tkn hub install task buildah
tkn hub install task openshift-client
```

Make sure that the **git-clone**, **buildah** and **openshift-client** tasks
are available in
the [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift)
using the following command:

```bash
oc get clustertask
```

8. To create [Tekton](https://tekton.dev) pipelines
   on [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift),
   use
   the following commands:

```bash
oc apply -f .openshift/tekton/pipelines/cba-pipeline.yml
oc apply -f .openshift/tekton/pipelines/cba-db-migration-revert-pipeline.yml
```

**Running Pipelines:**

1. To start CI/CD pipeline
   on [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift),
   use the following command:

```bash
tkn pipeline start cba-pipeline \
            -p repo-url=<GITHUB_REPO_URL> \
            -p branch=<BRANCH> \
            -p build-image=<DOCKER_IMAGE> \
            -p deploy-enabled=<DEPLOY_ENABLED> \
            -w name=<WORKSPAVCE_NAME>,claimName=<PVC_CLAIM_NAME> \
            -s pipeline \
            --showlog
```

- **GITHUB_REPO_URL** - URL of the GitHub repository
- **BRANCH** - name of the branch
- **DOCKER_IMAGE** - docker image with tag and registry, for example: `quay.
  io/username/cba:latest`
- **DEPLOY_ENABLED** - Enable/disable deployment step, for example: `true`
- **WORKSPACE_NAME** - name of the workspace specified in the pipeline yaml
  file, for example: `cba-pipeline-pvc`
- **PVC_CLAIM_NAME** - name of the PVC claim created for pipeline, for
  example: `cba-pipeline-pvc`

For example (Account microservice):

```bash
tkn pipeline start cba-pipeline \
            -p repo-url="https://github.com/hokushin118/account-service.git" \
            -p branch="main" \
            -p build-image=hokushin/account-service:latest \
            -p deploy-enabled=false \
            -w name=cba-pipeline-workspace,claimName=cba-pipeline-pvc \
            -s pipeline \
            --showlog
```

2. To start the database migration revert pipeline
   on [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift),
   use the following command:

```bash
tkn pipeline start cba-pipeline \
            -p repo-url=<GITHUB_REPO_URL> \
            -p branch=<BRANCH> \
            -p revision=<REVISION> \
            -w name=<WORKSPAVCE_NAME>,claimName=<PVC_CLAIM_NAME> \
            -s pipeline \
            --showlog
```

- **GITHUB_REPO_URL** - URL of the GitHub repository
- **BRANCH** - name of the branch
- **REVISION** - database migration revision id, for example: `1234abcd'
- **WORKSPACE_NAME** - name of the workspace specified in the pipeline yaml
  file, for example: `cba-pipeline-pvc`
- **PVC_CLAIM_NAME** - name of the PVC claim created for pipeline, for
  example: `cba-pipeline-pvc`

For example (Account microservice):

```bash
tkn pipeline start cba-pipeline \
            -p repo-url="https://github.com/hokushin118/account-service.git" \
            -p branch="main" \
            -p revision="1234abcd" \
            -w name=cba-pipeline-workspace,claimName=cba-pipeline-pvc \
            -s pipeline \
            --showlog
```

To make the pipeline ran successfully, run the following command:

```bash
tkn pipelinerun ls
```

You can check the logs of the last pipeline run with:

```bash
tkn pipelinerun logs --last
```

## Keycloak Identity and Access Management (IAM)

CBA microservices leverage [Keycloak](https://www.keycloak.org) for robust
authentication and authorization.

**Authentication and Authorization:**

* Microservices utilize [JWT](http://www.jwt.io) tokens for authentication and
  authorization.
* These tokens are issued by the Keycloak IAM server.
* Microservices validate the received [JWT](http://www.jwt.io) tokens.
* The [JWT](http://www.jwt.io) tokens are signed using the RSA256 algorithm.

**Realm Configuration:**

* The configuration for the `cba-dev` realm is stored in
  the `.infrastructure/keycloak/cba-dev-realm.json` file.
* This configuration is automatically imported when Keycloak is deployed using
  Docker Compose.

**Realm Roles (Development/Testing):**

* For development and testing purposes, the following realm roles are added:
    * `ROLE_USER`: Represents a standard user role.
    * `ROLE_ADMIN`: Represents an administrator role with elevated privileges.

**Accessing Keycloak:**

* **OpenID Configuration URL:
  ** [http://localhost:28080/realms/cba-dev/.well-known/openid-configuration](http://localhost:28080/realms/cba-dev/.well-known/openid-configuration) (
  Provides the OpenID Connect discovery document.)
* **Admin Console URL:** [http://localhost:28080](http://localhost:28080) (
  Provides a web interface for Keycloak administration.)

**Important Security Note:**

* **Default Admin Credentials:** The default administrator credentials (
  username: `admin`, password: `admin`) are provided for development purposes
  only. **Do not use these credentials in a production environment.**
* **Change Default Credentials:** Immediately change the default admin password
  after initial setup.

**Admin Console Credentials (Development Only):**

| **user name** | **password** |
|---------------|--------------|
| admin         | admin        |

## Prometheus Monitoring

CBA microservices are monitored using [Prometheus](https://prometheus.io), a
powerful open-source monitoring and alerting toolkit.

**Configuration:**

The [Prometheus](https://prometheus.io) configuration is located
in `.infrastructure/prometheus/prometheus.yml`. This file defines the metrics
scraping targets and other settings.

**Accessing Prometheus:**

You can access the [Prometheus](https://prometheus.io) web interface and
endpoints at the following URLs:

* **Prometheus Web UI:** [http://localhost:19090](http://localhost:19090) (
  Provides a graphical interface for querying and visualizing metrics.)
* **Metrics Endpoint:
  ** [http://localhost:19090/metrics](http://localhost:19090/metrics) (Displays
  the raw metrics data scraped by [Prometheus](https://prometheus.io).)
* **Targets Endpoint:
  ** [http://localhost:19090/targets](http://localhost:19090/targets) (Shows
  the status of the configured scraping targets.)

**Key Features:**

* [Prometheus](https://prometheus.io) allows you to monitor the health and
  performance of our microservices in real-time.
* You can use
  the [Prometheus Query Language (PromQL)](https://prometheus.io/docs/prometheus/latest/querying/basics/)
  to create custom queries and dashboards.
* Alerting rules can be configured to notify you of critical issues.

**Note:** Ensure [Prometheus](https://prometheus.io) is running (e.g., via
Docker Compose) to access these endpoints.

## Grafana Dashboards

CBA microservices utilize [Grafana](https://grafana.com) to create and manage
dashboards, visualizing metrics collected
by [Prometheus](https://prometheus.io) from our microservices.

**Purpose:**

[Grafana](https://grafana.com) dashboards provide real-time, visual
representations of key performance indicators (KPIs) and application health
metrics. They empower teams to:

* **Monitor Application Performance:** Track request rates, response times,
  error rates, and resource utilization.
* **Identify Anomalies:** Quickly detect unexpected behavior and potential
  issues.
* **Analyze Trends:** Gain insights into application performance over time.
* **Troubleshoot Issues:** Drill down into specific metrics to diagnose
  problems effectively.
* **Improve Visibility:** Offer a centralized view of application health for
  development, operations, and business teams.

**Configuration:**

[Grafana](https://grafana.com) configurations are managed via Docker Compose
and provisioning files, ensuring reproducible and version-controlled setups.

**Key Configuration Aspects:**

* **Data Source Provisioning:**
    * [Prometheus](https://prometheus.io) is configured as a data source using
      YAML provisioning files (
      e.g., `.infrastructure/metrics/grafana/provisioning/datasources/prometheus_ds.yaml`).
    * This ensures [Grafana](https://grafana.com) automatically connects to the
      correct [Prometheus](https://prometheus.io) instance upon startup.
    * The configuration specifies the Prometheus URL, access method (proxy or
      direct), and other relevant settings.
* **Dashboard Provisioning:**
    * [Grafana](https://grafana.com) dashboards are defined in JSON format and
      provisioned using YAML files (e.g., `all.yml`) located in
      the `.infrastructure/metrics/grafana/provisioning/dashboards` directory.
    * This enables version control and automated deployment of dashboards.
    * Dashboards visualize key metrics, including:
        * Request rates and response times.
        * HTTP status code distributions.
        * Resource utilization (CPU, memory).
        * Custom application metrics.
* **Dynamic Data Source Linking:**
    * In containerized environments (Docker Compose),
      the [Prometheus](https://prometheus.io) data source UID is dynamically
      injected into [Grafana](https://grafana.com) dashboard configurations
      using environment variables.
    * This ensures seamless connection to
      the [Prometheus](https://prometheus.io) instance without manual
      intervention.
* **Environment Variables:**
    * [Grafana](https://grafana.com) admin credentials are set via environment
      variables: `GF_ADMIN_USER` and `GF_ADMIN_PASSWORD`.
* **Docker Compose:**
    * [Grafana](https://grafana.com) runs as a Docker container, simplifying
      deployment and management.
    * Docker Compose links Grafana to the [Prometheus](https://prometheus.io)
      service, enabling seamless communication.
    * Volumes are used for persistent storage of Grafana data and provisioning
      files.

**Available Dashboards:**

* **Flask Monitoring Dashboard:**
    * Located in
      the `.infrastructure/metrics/grafana/provisioning/dashboards/flask-monitoring.json`
      file.
    * Focuses on real-time monitoring (3-second refresh, 5-minute time range).
    * Covers essential metrics like request rates, error rates, response times,
      and resource usage.
    * Utilizes common Prometheus queries for Flask applications.
    * Monitors performance of successful requests and errors.
    * Uses percentiles to monitor request latency.
    * Monitors CPU and memory usage.
    * Displays:
        * Rate of successful (HTTP 200) requests per second.
        * Rate of error requests (non-200 HTTP status codes) per second.
        * Total requests per minute, broken down by HTTP status code.
        * Average response time (seconds) for successful requests (HTTP 200)
          over 30 seconds.
        * 50th percentile (median) request duration (seconds) for successful
          requests (HTTP 200) over 30 seconds.
        * Resident memory usage (bytes) of the Flask application process.
        * Percentage of successful requests (HTTP 200) with response times
          under 250 milliseconds over 30 seconds.
        * 90th percentile request duration (seconds) for successful requests (
          HTTP 200) over 30 seconds.
        * CPU usage of the Flask application process.

## Audit with Kafka

CBA microservices utilize [Apache Kafka](https://kafka.apache.org) for robust,
scalable, and asynchronous audit logging. This architecture facilitates
centralized logging, real-time monitoring, and efficient data analysis, crucial
for debugging, security, and compliance. This setup is optimized for
development, testing, and local experimentation.

**Key Features:**

* **Asynchronous Logging:** [Apache Kafka](https://kafka.apache.org) enables
  non-blocking audit logging, minimizing performance impact on core
  microservices.
* **Centralized Logging:** Consolidated audit logs
  in [Apache Kafka](https://kafka.apache.org) simplify analysis and correlation
  across services.
* **Scalability:** [Apache Kafka](https://kafka.apache.org)'s distributed
  nature allows for handling increasing audit log volumes.
* **Real-time Monitoring:** Kafka Streams or similar tools can be used for
  real-time analysis of audit data.

**Prerequisites:**

* **Docker:** Ensure Docker is installed and running on your system.
* **Docker Compose:** Ensure Docker Compose is installed.
* **Environment Variables:** Configure necessary environment variables via
  a `.env` file or directly in your shell. See the
  example `docker-compose-kafka.yml` for required variables.

**Launching the Kafka Audit Environment:**

1. **Start Kafka Audit Services:**
    * Open your terminal and navigate to the directory
      containing `docker-compose-kafka.yml`.
    * Execute the following command to start
      the [Apache Zookeeper](https://zookeeper.apache.org)
      and [Apache Kafka](https://kafka.apache.org) containers:

        ```bash
        docker compose -f docker-compose-kafka.yml up -d
        ```

        * `-f docker-compose-kafka.yml`: Specifies the Docker Compose file to
          use.
        * `up`: Starts the containers.
        * `-d`: Runs the containers in detached mode (background).

2. **Verify Service Status:**
    * Confirm the services are running correctly:

        ```bash
        docker ps
        ```

        * This command lists running Docker containers. You should see the
          Zookeeper and Kafka containers.

3. **View Container Logs:**
    * To view the logs of a specific container (e.g., Kafka):

        ```bash
        docker logs kafka-1
        ```

        * Replace `kafka-1` with the actual container name.

**Accessing Kafka:**

* **From within the Kafka container:** Use `localhost:9093`
  for `kafka-console-consumer.sh` and related commands.
* **From other Docker containers on the same network:** Use `kafka-1:9092` (or
  the appropriate internal port).

**Interacting with Kafka:**

* **Using `kafka-console-consumer.sh` (within the Kafka container):**

    ```bash
    docker exec -it kafka-1 /opt/bitnami/kafka/bin/kafka-console-consumer.sh \
      --bootstrap-server localhost:9093 --topic audit-events --from-beginning
    ```

**Important Notes:**

* **Environment Variables:** Use a `.env` file for managing environment
  variables.
* **Production:** This setup is for development. For production environments,
  consider a multi-node Kafka cluster and proper security configurations.
