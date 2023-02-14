# CI/CD Microservices App
Repository hosting a Google Kubernetes Engine (GKE) microservices app and CI/CD pipeline

## Application Architecture
This application is based on the freeCodeCamp guide, [Learn Kubernetes in Under 3 Hours: A Detailed Guide to Orchestrating Containers](https://www.freecodecamp.org/news/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882/), by [Rinor Maloku](https://www.freecodecamp.org/news/author/rinormaloku/)

<img src="./diagrams/application-architecture.drawio.svg">

## CI/CD Solution Architecture

## Running the Applicaiton
The steps outlined here are based on the hands-on [lab](https://www.freecodecamp.org/news/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882/) from freeCodeCamp
### Running the Local App
> NOTE: for local app, navigate to `App.js` in `sa-frontend` and comment out line 24 and uncommment line 23.
1. Navigate to `sa-frontend` and execute the following command
   ```
   npm install
   ```
   > OPTIONAL: start the frontend app via the below command
   ```
   npm start
   ```
2. Build the app
   ```
   npm run build
   ```
3. Copy the contents of `sa-frontend/build` to an nginx server at `[your_nginx_installation_dir]/html`
4. Navigate to `sa-webapp` and execute the following command
   ```
   mvn install
   ```
   > OPTIONAL: start the webapp via the below command  (first navigate to `/target`)
   ```
   java -jar sentiment-analysis-web-0.0.1-SNAPSHOT.jar --sa.logic.api.url=http://localhost:5000
   ```
5. Navigate to `sa-logic/sa` and execute the followign commands
    ```
    python -m pip install -r requirements.txt
    ```
    ```
    python -m textblob.download_corpora
    ```
    > OPTIONAL: start the logic app via the below command
    ```
    python sentiment_analysis.py
    ```
6. To see the app in aciton (local), run the image of nginx which has the updated `/html` fodler and execute the OPTIONAL commands in steps 5 and 6 in sparate terminal sessions

### Running the local Dockerized Microservices
For this section, it is assumed that the reader has the Docker Desktop application and a Docker Hub account.
1. Build all the Docker images
   > Logic Service
   ```
   docker build -f Dockerfile -t <DOCKER_HUB_USERNAME>/sentiment-analysis-logic .
   ```
   > WebApp Service
   ```
   docker build -f Dockerfile -t <DOCKER_HUB_USERNAME>/sentiment-analysis-web-app .
   ```
   > Frontend Service
   ```
   docker build -f Dockerfile -t <DOCKER_HUB_USERNAME>/sentiment-analysis-frontend .
   ```
2. Push all images to your Docker Hub
   ```
   docker push <DOCKER_HUB_USERNAME>/sentiment-analysis-<SERVICE>
   ```
   > SERVICE being one of {`logic`, `web-app`, `frontend`}
3. Pull the images from Docker Hub
   ```
   docker pull <DOCKER_HUB_USERNAME>/sentiment-analysis-<SERVICE>
   ```
   > SERVICE being one of {`logic`, `web-app`, `frontend`}
4. Run the Logic Service
   ```
   docker run -d -p 5050:5000 <DOCKER_HUB_USERNAME>/sentiment-analysis-logic
   ```
5. Run the WebApp service
   ```
   docker run -d -p 8080:8080 -e SA_LOGIC_API_URL='http://<CONTAINER_IP>:5000' <DOCKER_HUB_USERNAME>/sentiment-analysis-web-app
   ```
   > NOTE: to retrieve the `<CONTAINER_IP>`, run `docker container list` followed by `docker inspect <CONTAINER ID>` and search the response for the container IP address. 
6. Run the Frontend Service
   ```
   docker run -d -p 80:80 <DOCKER_HUB_USERNAME>/sentiment-analysis-frontend
   ```

### Deploying App to Local Kubernetes Cluster <a name="deploy"></a>
It is assumed the reader has installed minikube and that the images for `sa-webapp` and `sa-logic` have been built.
1. Execute `minikube start`
2. Navigate to `resrouce-manifests` and execute the below command to deploy the logic service
   ```
   kubectl apply -f sa-logic-deployment.yaml
   ```
3. Deploy the logic service entry point service
   ```
   kubectl apply -f service-sa-logic.yaml
   ```
4. Deploy the webapp service
   ```
   kubectl apply -f sa-web-app-deployment.yaml
   ```
5. Deploy the webapp loadbalancer service
   ```
   kubectl apply -f service-sa-web-app-lb.yaml
   ```
6. Run the webapp load balancer service and identify the URL via `minikube service sa-web-app-lb --url`
7. Navigate to `sa-frontend` and modify `src/App.js` on line 23 by replacing the URL in the `fetch(...)` with the URL from step 7
8. Rebuild and deploy the frontend
   > In `sa-frontend`
   ```
   docker build -f Dockerfile -t boszin/sentiment-analysis-frontend .
   docker push boszin/sentiment-analysis-frontend
   ```
   > In `resource-manifests`
   ```
   kubectl apply -f sa-frontend-deployment.yaml
   ```
9. Deploy the frontend loadbalancer service
   ```
   kubectl apply -f service-sa-frontend-lb.yaml
   ```
   > OPTIONAL: check your deployement via `minikube service sa-frontend-lb`

## Deploying GKE via Terraform
These steps are based on the guide [here](https://developer.hashicorp.com/terraform/tutorials/kubernetes/gke)
1. Navigate to `terraform/terraform.tfvars` and set `project_id` to your Google Cloud Project
2. In the `terraform` folder, execute `terraform init`
3. Execute `terraform apply`, review the chagnes and if all looks well, type `yes`
4. Configure your `kubectl` via the below command
   ```
   gcloud container clusters get-credentials <kubernetes_cluster_name> --region <region>
   ```
   > `<kubernetes_cluster_name>` and `<region>` should have been an output from step 4.
5. To tear down the deployement (and avoid incurred costs) run `terraform destroy` when done experimenting

## Deploying Microservices App to GKE (manual)
Ensure `gcloud` and `gke-gcloud-auth-plugin` have been installed and are configured
1. Navigate to the `resource-manifests` fodler and execute the same commands as [Deploying App to Local Kubernetes Cluster](#deploy)
2. For step 6 in the above reference, replace `minikube service list` with `kubectl describe services sa-web-app-lb` to get the URL.

## Deploying Microservices App to GKE (CI/CD)
This repository is enabled with GitHub workflows (see `.github/workflows`) which automate the manual tasks and commands performed above. The workflows are based on the guide [here](https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-google-kubernetes-engine).
> NOTE 1: Changes to the WebApp Microservice will require an update to the Frontend service. Line 23 in `App.js` must be updated with the URL of the WebApp Loadbalancer upon re-deploy; and the frontend must be rebuilt and redeployed as a result. Parametarizing this URL is a future enhancement.

> NOTE 2: Ensure that the account used for your CI/CD pipelines has the following permissions:
> - Compute Admin
> - Compute Network Admin
> - Kubernetes Engine Admin
> - Service Account User
> - Storage Admin

Other references:
- https://cloud.google.com/kubernetes-engine/docs/archive/using-container-image-digests-in-kubernetes-manifests#using_kustomize
- https://cloud.google.com/blog/products/identity-security/enabling-keyless-authentication-from-github-actions
- https://github.com/google-github-actions/auth#setup

### Blue/Green Deployement Scheme
To test out the [Blue/Green deployment scheme](https://cloud.google.com/architecture/implementing-deployment-and-testing-strategies-on-gke#perform_a_bluegreen_deployment), perform the following:
1. In `sa-frontend/App.js` change line 49 by adding "(Blue)" to the title
2. In `resource-manifests-gke/sa-frontend/deployment.yml` edit the deployement to be named `sa-frontend-blue`
3. In the same location, ensure the `service.yml` points to the `sa-frontend-blue` selector
4. Push the code to build and deploy the deployement and service
5. Update line 49 in `sa-frontend/App.js` from "Blue" to "Green"
6. In `resource-manifests-gke/sa-frontend/deployment.yml` edit the deployement to be named `sa-frontend-green` and push your code to automatically build the new deployement
7. When you are ready, open the frontend page and you will see it has "Blue" in the title
8. In a new terminal, navigate to `resource-manifests-gke/sa-frontend` and modify `service.yml` to point to the `sa-frontend-green` selector
9. Execute `kubectl apply -f service.yml`
10. Go back to your browser and refresh the page a few times. After a few seconds, you should now see that the title went from having "Blue" to having "Green"

## BONUS: Service Mesh on GKE
To enable the Managed Anthos Service Mesh, follow the steps [here](https://cloud.google.com/service-mesh/docs/unified-install/install-anthos-service-mesh-command)
> Note: the GKE deployment for this application can be enabled with Workload Identity via Terraform (see [here](https://registry.terraform.io/modules/terraform-google-modules/kubernetes-engine/google/9.2.0/submodules/workload-identity)); however, the functionality was ommited for the original inention of the project.
