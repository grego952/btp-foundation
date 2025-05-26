---
parser: v2
auto_validation: true
time: 40
tags: [ tutorial>intermediate, topic>cloud, software-product>sap-business-technology-platform]
primary_tag: software-product>sap-btp--kyma-runtime
---

# Deploy the SAPUI5 Frontend in SAP BTP, Kyma Runtime
<!-- description --> Develop and deploy the SAPUI5 frontend app in SAP BTP, Kyma runtime.

## Prerequisites
  - [Docker](https://www.docker.com/) installed with a valid public account
  - [kubectl configured to kubeconfig downloaded from SAP BTP, Kyma runtime](cp-kyma-download-cli)
  - [Git](https://git-scm.com/downloads)
  - [Node.js](https://nodejs.org/en/download/)
  - [UI5 Tooling](https://sap.github.io/ui5-tooling/)
  - [Deploy a Go MSSQL API Endpoint in SAP BTP, Kyma Runtime](cp-kyma-api-mssql-golang) tutorial completed

## You will learn
  - How to configure and build SAPUI5 Docker image
  - How to deploy the SAPUI5 Docker image to SAP BTP, Kyma runtime

## Intro
This tutorial expects that the [Deploy a Go MSSQL API Endpoint in SAP BTP, Kyma Runtime](cp-kyma-api-mssql-golang) tutorial has been completed and relies on the database running within SAP BTP, Kyma runtime.

Deploying the SAPUI5 Docker image to SAP BTP, Kyma runtime includes:

- A Kubernetes Deployment of the frontend image with the ConfigMap mounted to a volume
- A Kubernetes ConfigMap containing the URL to the backend API
- A Kubernetes Service used to expose the application to other Kubernetes resources
- A Kyma APIRule to expose the frontend application to the Internet

---

### Clone the Git repository

1. Go to [kyma-runtime-extension-samples](https://github.com/SAP-samples/kyma-runtime-extension-samples). This repository contains a collection of Kyma sample applications which will be used during the tutorial.

2. Use the **Code** button to choose one of the options to download the code locally, or simply run the following command in your CLI at your desired folder location:

    ```Shell/Bash
    git clone https://github.com/SAP-samples/kyma-runtime-extension-samples
    ```

### Explore the sample

1. Open the `frontend-ui5-mssql` directory in your desired editor.

2. Explore the content of the sample.

    You can find many directories that relate to the UI5 application.

    The `docker` directory contains the Dockerfile used to generate the Docker image. The image is built in two stages to create a small image. In the first stage, a `NodeJS` image is used, which copies the related content of the project into the image and builds the application. The built application is then copied into an `nginx` image with the default setup exposing the application on port 80.

    Within the `k8s` directory, you can find the Kubernetes/Kyma resources you will apply to your SAP BTP, Kyma runtime.

### Run the application locally

1. Within the `frontend-ui5-mssql` directory, run the following command using your CLI to install the application dependencies:

    ```Shell/Bash
    npm install
    ```

2. Run the following command to get the SAP BTP, Kyma runtime cluster domain:

    ```bash
    kubectl get gateway -n kyma-system kyma-gateway \
            -o jsonpath='{.spec.servers[0].hosts[0]}'
    ```

    Result should look like this:

    ```bash
    *.<xyz123>.kyma.ondemand.com
    ```

3. Within the project, open the `webapp/config.json` file and adjust `API_URL` to match the cluster domain.

    ```Text/Javascript
    {
      "API_URL": "https://api-mssql-go.*******.kyma.ondemand.com"
    }
    ```

4. To start the application, run the following command, which should automatically load the application in your browser.

    ```Shell/Bash
    npm run-script start
    ```
    > Pressing `control-c` will stop the running application.

5. After you tested the application locally, go to the `api-mssql-go/k8s` directory, and open `apirule.yaml`.
   
6. Remove the `exact: http://localhost:8080` line, and apply the APIRule.

    ```Shell/Bash
    kubectl -n dev apply -f ./k8s/apirule.yaml
    ```

### Build Docker image

Run the following commands from the `frontend-ui5-mssql` directory using your CLI. Make sure to replace the value of `<your-docker-id>` with your Docker account ID.

1. To build the Docker image, run this command:

    ```Shell/Bash
    docker build -t <your-docker-id>/fe-ui5-mssql -f docker/Dockerfile .
    ```

2. To push the Docker image to your Docker repository, run this command:

    ```Shell/Bash
    docker push <your-docker-id>/fe-ui5-mssql
    ```

### Apply resources to SAP BTP, Kyma runtime

You can find the resource definitions in the `k8s` folder. If you performed any changes in the configuration, these files may also need to be updated. The folder contains the following files that are relevant to this tutorial:

- `apirule.yaml`: defines the API endpoint which exposes the application to the Internet. This endpoint does not define any authentication access strategy and should be disabled when not in use.  
- `configmap.yaml`: defines `API_URL`, which represents the endpoint used for the orders API. It must be adjusted to reflect the domain of SAP BTP, Kyma runtime.
- `deployment.yaml`: defines the Deployment definition for the SAPUI5 application as well as a service used for communication. This definition references `configmap.yaml` by name. It is used to overwrite `webapp/config.json` of the application.

1. Within the project, open the `k8s/configmap.yaml` file and adjust `API_URL` to match the cluster domain:

    ```
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: fe-ui5-mssql
      labels:
        app: fe-ui5-mssql
    data:
      config.json: |-
        {
          "API_URL": "https://api-mssql-go.*******.kyma.ondemand.com"
        }
    ```

2. Apply the ConfigMap:

    ```Shell/Bash
    kubectl -n dev apply -f ./k8s/configmap.yaml
    ```

3. In `deployment.yaml`, adjust the value of `spec.template.spec.containers.image` to use your Docker image. Apply the Deployment:

    ```Shell/Bash
    kubectl -n dev apply -f ./k8s/deployment.yaml
    ```

4. Apply the APIRule:

    ```Shell/Bash
    kubectl -n dev apply -f ./k8s/apirule.yaml
    ```

### Open the UI application

To access the application, you can use the APIRule created in the previous step.

1. Open Kyma dashboard.

2. From the menu, choose **Namespaces**, and go to the `dev` namespace.

3. Choose **Discovery and Network > API Rules**.

4. Choose the **Host** entry for the **fe-ui5-mssql** APIRule to open the application in the browser. This should be similar to:
`https://fe-ui5-mssql.*******.kyma.ondemand.com`

---