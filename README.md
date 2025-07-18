# TW3 CI/CD


Hello! This is the main repo for the TW3 case study. Here we will answer the questions of the case study and show the project and the CI/CD workflow. This repository is meant to serve as a way for you to test the app youself very easily, however, it is also meant to illustrate later how the CI/CD srategy will be. 

As you might know already, the frontend and the backend are isolated and developed in different repositories (as they should), and are found in the following repositories:

front
   ```bash
   https://github.com/JCLMantilla/TW3_frontend.git
   ```
back
   ```bash
   https://github.com/JCLMantilla/TW3_backend.git
   ```

The working docker image of both repositories are aleady published in the github container registry (you can see them in my packages https://github.com/JCLMantilla?tab=packages). As a part of the CI/CD workflow, These images are automatically created on a new release by a custom github action, then the lastest image for both frontend and backend are ready to be pulled in deployment in the cloud.


We have a ```prod.docker-compose.yml``` file that will be in charge of pulling the lastest images and building the app. Here you can also change ports if needed in production. Keep in mind that you must include a ```.env``` file in the root with the following variables:

   ```bash
    # API keys
    QWEN_API_KEY = "api_key"
    SCRAPINGBEE_API_KEY = "api_key"
   ```

## Startup of the program

To start the program just run ```docker compose up ```, you will able to acess the chat in ```http://localhost:3000``` on your browser and test it if you set up valid API keys. Here is how it works:




# How the app works 




I implemented the *frontend* using React and Vite in a Docker container using a light node image. The visual interfaces was done with help of ChatGPT.

The backend was also built using Docker with a light Python image in order to reduce container size and build cost (in order to reduce costs and build time). This app was done using FastAPI.

In order to explain how the backend workds, lets divide ito two sections: *Search Engine* and *Agent*

### *Search engine*

This built-from-scratch search engine uses the free tier of ScrapingBee to do a google search using a query (which will be provided by the agent). Then ScrapingBee extracts the raw html of the first *n* pages (3 in this case), and in each wepage we retrieve the first *k* big and meaningful chunks of information by cleaning up the html extensively using readability, beautiful boup and some regex. This cleanup removes tags and other irelevant info from the DOM, and deals with segmentation by cleaning and replacing line-skips. This search engine is completely async


### *Agent*

We defined a LangChain Agent that was access to a search engine as a LangChain Tool. This agent is exposed using FastAPI, and uses a custom wraper of the *"qwen2.5-32b-instruct"* model by using the ChatOpenAI class of LangChain (since qwen models are usable through the OpenAI SDK already).

This agent is *asyncronous*, has memory and it is initialized whenever the app started. *This is a very important assumption* since in production we would like for each user to have their own agent, and not share the same one with every other user. Also, if we want to horizontally scale we must have each container serving many users at the same time.

## *Error handling*

The backend logs inside the container are logged using the standard logging module with their exception details. These logs must be catched in production by log manager in order to *monitor* and help debugging. I normally use *BugSnag*, which is easily integrated as a lets me FastAPI middleware. For this project I could not implement the BugSnag real-time monitoring since I no longer have my BugSnag account.

However, the app has a nice error handling logic to avoid exposing logs into the users and to avoid breaking the app. If the search engine fails due to lack of credits but the agent is still running, the agent will respond that he could not reach the information. If the whole backend fails due to isufficient credits on the LLM, the front end will always respond that it could not reach the backend. This message will also be displayed by the frontend if the backend is completely disconnected.




# CI/CD

This project was designed to be used in a Kubernetes-based infrastructure. Since we can pull the lastest images of frontend and backend, we can connect to a Kubernetes cluster create/update our Kubernetes Deployment and Service for both frontend and backend very easily in the command line  using ```kubectl```. We can define replicas in our Kubernetes service in order to scale base on our load!

This is how the CI/CD Pipeline Works:

- We Implement a new feature and run tests before a pull request
- Commit & Push
- Docker images are built for both the frontend and backend on release.
- Built images are tagged and pushed to a container registry.
- Once images are built and pushed, the Kubernetes deployment step applies updated manifests to the Kubernetes cluster.
- Kubernetes Deployments automatically roll out updated containers.
- Health checks ensure that bad deployments are rolled back safely.
- Logs and metrics are collected for observability using Bugsnag (My to-go tool to monitoring).


This setup allows for safe, repeatable, and scalable deployments in any Kubernetes-based infrastructure.


# Azure CI/CD

I have never used Azure services however, I know that my CI/CD practice can be implemented and it is a standard practice usingw the following services:

Azure Kubernetes Service (AKS): Managed Kubernetes cluster
Azure Container Registry (ACR): Docker image registry
Azure DevOps Pipelines instead of GitHub Actions: CI/CD platform
Ingress Controller (NGINX or Azure Application Gateway): For routing frontend traffic
Secrets via Azure Key Vault: For environment configs, API keys, etc.

We can also enhance our DevOps by:

Using Helm for versioned deployments
Adding Staging environments




