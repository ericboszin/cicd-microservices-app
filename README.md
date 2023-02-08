# CI/CD Microservices App
Repository hosting a Google Kubernetes Engine (GKE) microservices app and CICD pipeline

## Application Architecture
This application is based on the freeCodeCamp guide, [Learn Kubernetes in Under 3 Hours: A Detailed Guide to Orchestrating Containers](https://www.freecodecamp.org/news/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882/), by [Rinor Maloku](https://www.freecodecamp.org/news/author/rinormaloku/)


## CI/CD Solution Architecture

## Running the Local App
1. Clone this repository
2. Navigate to `sa-frontend` and execute the following command
   ```
   npm install
   ```
   > OPTIONAL: start the frontend app via the below command
   ```
   npm start
   ```
3. Build the app
   ```
   npm run build
   ```
4. Copy the contents of `sa-frontend/build` to an nginx server at `[your_nginx_installation_dir]/html`
5. Navigate to `sa-webapp` and execute the following command
   ```
   mvn install
   ```
   > OPTIONAL: start the webapp via the below command  (first navigate to `/target`)
   ```
   java -jar sentiment-analysis-web-0.0.1-SNAPSHOT.jar --sa.logic.api.url=http://localhost:5000
   ```
6. Navigate to `sa-logic/sa` and execute the followign commands
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
7. To see the app in aciton (local), run the image of nginx which has the updated `/html` fodler and execute the OPTIONAL commands in steps 5 and 6 in sparate terminal sessions

## Running the local Dockerized microservices
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
   docker run -d -p 5050:5000 <DOCKER_HUB_USERNAME>/sentiment-analysis-frontend
   ```