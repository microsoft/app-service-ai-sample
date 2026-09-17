# App Service AI sample

This repository contains a Python Quart chat app for Azure App Service. The
app uses managed identity to call an Azure OpenAI model deployment and streams
responses to a browser UI.

The application source is adapted from
[Azure-Samples/openai-chat-app-quickstart](https://github.com/Azure-Samples/openai-chat-app-quickstart).
The original sample targeted Azure Container Apps. This repository contains
App Service deployment files instead. It does not include the old Container
Apps deployment modules.

## What is included

* `app.py` and `quartapp/` contain the Quart backend and browser UI.
* `gunicorn.conf.py` defines the App Service production server settings.
* `infra/` provisions an App Service plan, a Linux web app, managed identity,
  Azure OpenAI access, and optionally an Azure OpenAI resource and model
  deployment.
* `azure.yaml` connects the `web` service to the App Service resource for
  Azure Developer CLI deployments.

## Required configuration

The app reads these settings:

| Setting | Use |
| --- | --- |
| `AZURE_OPENAI_ENDPOINT` | Azure OpenAI resource endpoint, such as `https://example.openai.azure.com/`. |
| `AZURE_OPENAI_CHAT_DEPLOYMENT` | Azure OpenAI deployment name. This can differ from the model name. |
| `AZURE_TENANT_ID` | Tenant used by Azure Developer CLI credentials during local development. |
| `AZURE_CLIENT_ID` | Optional client ID for a user-assigned managed identity. The included Bicep uses a system-assigned identity and does not set this value. |
| `RUNNING_IN_PRODUCTION` | Reduces application log verbosity when set. |

The deployed identity needs the `Cognitive Services OpenAI User` role on the
Azure OpenAI resource. The included Bicep creates that role assignment.

## Deployment paths

### Existing App Service or portal-created resources

The application can run on an existing Linux App Service app. This repository
does not contain a portal create template or any unreleased portal
infrastructure integration. Before deploying the source, configure the App
Service identity, role assignment, app settings, Python runtime, and startup
command described in [the existing-resource guide](docs/deploy-existing.md).

This path requires an Azure OpenAI resource and model deployment that already
exist.

### Azure Developer CLI

The `azd` path provisions the App Service resources and deploys the
application:

```shell
azd auth login
azd up
```

By default, the Bicep template also creates an Azure OpenAI resource and model
deployment. To reuse an existing Azure OpenAI resource, set the values in
[the existing-resource guide](docs/deploy-existing.md) before running
`azd up`.

These resources can incur Azure charges. Model and regional availability also
vary by subscription.

## Run locally

Create an Azure OpenAI resource and model deployment, or reuse existing ones.
Clone the official repository, copy `.env.sample` to `.env`, and set the
endpoint, deployment name, and tenant. Then authenticate and run the app:

```shell
git clone https://github.com/microsoft/app-service-ai-sample.git
cd app-service-ai-sample
python -m venv .venv
pip install -r requirements.txt
azd auth login
python -m quart --app app run --port 50505 --reload
```

On Windows PowerShell, activate the virtual environment with
`.\.venv\Scripts\Activate.ps1`. On macOS or Linux, use
`. .venv/bin/activate`.

Open <http://localhost:50505>.

## Data flow and telemetry

The browser sends chat history to this app. The backend sends that content to
the configured Azure OpenAI endpoint with `store=false`, then streams response
events back to the browser. See [Data flow and privacy](docs/data-flow.md) for
the full technical description.

The application code does not include an analytics SDK, Application Insights
SDK, or custom Microsoft telemetry. Azure App Service and Azure OpenAI may
produce platform and service logs according to the resource owner's Azure
configuration. The browser also downloads UI libraries from jsDelivr.

## Security

Report security vulnerabilities privately as described in
[SECURITY.md](SECURITY.md). Do not report vulnerabilities through public
GitHub issues.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). This project follows the
[Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).

## License and attribution

This project is licensed under the [MIT License](LICENSE.TXT). See
[NOTICE](NOTICE) for source attribution and dependency information.

Trademarks: This project may contain trademarks or logos for projects,
products, or services. Authorized use of Microsoft trademarks or logos is
subject to and must follow
[Microsoft's Trademark and Brand Guidelines](https://www.microsoft.com/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project
must not cause confusion or imply Microsoft sponsorship.
