# Video Eval Service

## Description

This service is responsible for evaluating the video type answer submissions using Azure OpenAI
services. This service gets triggered by consumer backend service when a video type answer is
submitted by the user. The service then evaluates the answer and returns the feedback back to the
consumer backend service.

## Swagger Link

https://video-eval-service-prod.azurewebsites.net/docs

## Infra

### Hosting
- The project is hosted on **Azure App Service**.
- The project is **dockerized** and the images gets pushed to the **Azure Container Registry**.
- The project gets deployed using the **Azure App Service Portal**.

### Logging
- The FastAPI logs are written to the folder `logs/`.
- The logs are then read by Filebeat and sent to our self-hosted ELK stack (http://elasticsearch.internal.crio.do:9200).
- The logs can be viewed in Kibana (http://kibana.internal.crio.do:5601/).
- The logs are stored in the index `video-eval-service-prod`.
- The Filebeat configuration file can be found at `filebeat.yml`.

### Azure OpenAI
- This project uses Azure OpenAI transcription and chat completions APIs to evaluate the user's answer.
- The Azure OpenAI API key & the secret token is stored in the `crio_vault`.
- We are using the [WhipserOpenAI](https://portal.azure.com/#@rathinacriodo.onmicrosoft.com/resource/subscriptions/761a40ba-9ee7-4c99-b035-7147d1bdbfd8/resourceGroups/CrioOpenAI/providers/Microsoft.CognitiveServices/accounts/WhipserOpenAI/overview) resource in Azure OpenAI portal.
- We are currently using `tech-hr-eval-gpt-35-0125` deployment for the evaluation, which is a GPT-3.5 model version 0125.
- We are currently using `test` deployment for the transcription, which is a Whisper model version 001.

## Deployment

1. Use `./build.sh` to build the docker image and push it to the Azure Container Registry.
2. Use the Azure App Service portal to deploy the newly pushed image.
    1. Go to the [Azure App Service Portal](https://portal.azure.com/#@rathinacriodo.onmicrosoft.com/resource/subscriptions/761a40ba-9ee7-4c99-b035-7147d1bdbfd8/resourceGroups/CRIO-PROD-RG/providers/Microsoft.Web/sites/video-eval-service-prod/appServices).
    2. Click on `Deployment > Deployment slots` from the left sidebar.
    3. Click on the `+ Add` button to add a new deployment slot.
    4. Set the `Name` of the slot to `staging` and set `Clone settings from` to `video-eval-service-prod`.
    5. Now click on the `Add` button below to create the slot.
    6. After the slot is created, click on the `Swap` button to swap the staging slot with the production slot.
    7. Click on the `Start Swap` button to start the swap process.
    8. Now the new image will be deployed to the production slot.
3. The service is now deployed and ready to use.

## Flows

### Evaluation Trigger

![Evaluation Trigger](assets/evaluation-trigger.png)

### Evaluation

![Evaluation](assets/evaluation.png)

## Project File Structure

```bash
api/    # API routes
  ├── v[0-9]+/    # Version wise api routes
  ├── ...
  └── dependencies.py    # Dependency injection
assets/    # Project assets
  └── ...
core/
  ├── chains/    # LangChain chains for evaluating user's answer
  ├── exceptions/    # Custom exceptions
  ├── schemas/    # Pydantic schemas
  ├── services/    # Core services
  ├── utils/    # Utility functions
  ├── ...
  └── settings.py    # Project configuration settings
logs/    # Log files
  └── ...
tests/    # Test cases
  └── ...
...
build.sh    # Build script for building the docker image and pushing it to the Azure Container Registry
Dockerfile    # Dockerfile for building the image
...
filebeat.yml    # Filebeat configuration file (for more info see https://www.elastic.co/beats/filebeat)
gunicorn.conf.py    # Gunicorn configuration file
main.py    # FastAPI application entry point
poetry.lock    # Poetry lock file (for more info see https://python-poetry.org/docs/)
pyproject.toml    # Poetry project file (for more info see https://python-poetry.org/docs/)
README.md    # This file!
requirements.txt    # Python requirements file (for compatibility with Azure App Service)
run.sh    # Script to run the application locally
supervisord.conf    # Supervisor configuration file (for more info see http://supervisord.org/)
```
