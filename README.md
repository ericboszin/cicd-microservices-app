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
   > OPTIONAL: start the webapp via the below command 
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

