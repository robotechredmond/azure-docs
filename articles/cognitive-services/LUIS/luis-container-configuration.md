---
title: Docker container settings
titleSuffix: Language Understanding - Azure Cognitive Services
description: The LUIS container runtime environment is configured using the `docker run` command arguments. LUIS has several required settings, along with a few optional settings.   
services: cognitive-services
author: diberry
manager: cgronlun
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual 
ms.date: 01/22/2019
ms.author: diberry 
---

# Configure Language Understanding docker containers 

The Language Understanding (LUIS) container runtime environment is configured using the `docker run` command arguments. LUIS has several required settings, along with a few optional settings. Several [examples](#example-docker-run-commands) of the command are available. The container-specific settings are the input [mount settings](#mount-settings) and the billing settings. 

Container settings are [hierarchical](#hierarchical-settings) and can be set with [environment variables](#environment-variable-settings) or docker [command-line arguments](#command-line-argument-settings).

## Configuration settings

This container has the following configuration settings:

|Required|Setting|Purpose|
|--|--|--|
|Yes|[ApiKey](#apikey-setting)|Used to track billing information.|
|No|[ApplicationInsights](#applicationinsights-setting)|Allows you to add [Azure Application Insights](https://docs.microsoft.com/azure/application-insights) telemetry support to your container.|
|Yes|[Billing](#billing-setting)|Specifies the endpoint URI of the service resource on Azure.|
|Yes|[Eula](#eula-setting)| Indicates that you've accepted the license for the container.|
|No|[Fluentd](#fluentd-settings)|Write log and, optionally, metric data to a Fluentd server.|
|No|[Http Proxy](#http-proxy-credentials-settings)|Configure an HTTP proxy for making outbound requests.|
|No|[Logging](#logging-settings)|Provides ASP.NET Core logging support for your container. |
|Yes|[Mounts](#mount-settings)|Read and write data from host computer to container and from container back to host computer.|

> [!IMPORTANT]
> The [`ApiKey`](#apikey-setting), [`Billing`](#billing-setting), and [`Eula`](#eula-setting) settings are used together, and you must provide valid values for all three of them; otherwise your container won't start. For more information about using these configuration settings to instantiate a container, see [Billing](luis-container-howto.md#billing).

## ApiKey setting

The `ApiKey` setting specifies the Azure resource key used to track billing information for the container. You must specify a value for the ApiKey and the value must be a valid key for the _Language Understanding_ resource specified for the [`Billing`](#billing-setting) configuration setting.

This setting can be found in the following places:

* Azure portal: **Language Understanding's** Resource Management, under **Keys**
* LUIS portal: **Keys and Endpoint settings** page. 

Do not use the starter key or the authoring key. 

## ApplicationInsights setting

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## Billing setting

The `Billing` setting specifies the endpoint URI of the _Language Understanding_ resource on Azure used to meter billing information for the container. You must specify a value for this configuration setting, and the value must be a valid endpoint URI for a _Language Understanding_ resource on Azure.

This setting can be found in the following places:

* Azure portal: **Language Understanding's** Overview, labeled `Endpoint`
* LUIS portal: **Keys and Endpoint settings** page, as part of the endpoint URI.

|Required| Name | Data type | Description |
|--|------|-----------|-------------|
|Yes| `Billing` | String | Billing endpoint URI<br><br>Example:<br>`Billing=https://westus.api.cognitive.microsoft.com/luis/v2.0` |

## Eula setting

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## Fluentd settings


[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## Http proxy credentials settings

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## Logging settings
 
[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]


## Mount settings

Use bind mounts to read and write data to and from the container. You can specify an input mount or output mount by specifying the `--mount` option in the [docker run](https://docs.docker.com/engine/reference/commandline/run/) command. 

The LUIS container doesn't use input or output mounts to store training or service data. 

The exact syntax of the host mount location varies depending on the host operating system. Additionally, the [host computer](luis-container-howto.md#the-host-computer)'s mount location may not be accessible due to a conflict between permissions used by the docker service account and the host mount location permissions. 

The following table describes the settings supported.

|Required| Name | Data type | Description |
|-------|------|-----------|-------------|
|Yes| `Input` | String | The target of the input mount. The default value is `/input`. This is the location of the LUIS package files. <br><br>Example:<br>`--mount type=bind,src=c:\input,target=/input`|
|No| `Output` | String | The target of the output mount. The default value is `/output`. This is the location of the logs. This includes LUIS query logs and container logs. <br><br>Example:<br>`--mount type=bind,src=c:\output,target=/output`|

## Hierarchical settings

[!INCLUDE [Container shared configuration hierarchical settings](../../../includes/cognitive-services-containers-configuration-shared-hierarchical-settings.md)]


## Example docker run commands

The following examples use the configuration settings to illustrate how to write and use `docker run` commands.  Once running, the container continues to run until you [stop](luis-container-howto.md#stop-the-container) it.


* **Line-continuation character**: The docker commands in the following sections use the back slash, `\`, as a line continuation character. Replace or remove this based on your host operating system's requirements. 
* **Argument order**: Do not change the order of the arguments unless you are very familiar with docker containers.

Replace {_argument_name_} with your own values:

| Placeholder | Value | Format or example |
|-------------|-------|---|
|{ENDPOINT_KEY} | The endpoint key of the trained LUIS application. |xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx|
|{BILLING_ENDPOINT} | The billing endpoint value is available on the Azure portal's Language Understanding Overview page.|https://westus.api.cognitive.microsoft.com/luis/v2.0|

> [!IMPORTANT]
> The `Eula`, `Billing`, and `ApiKey` options must be specified to run the container; otherwise, the container won't start.  For more information, see [Billing](luis-container-howto.md#billing).
> The ApiKey value is the **Key** from the Keys and Endpoints page in the LUIS portal and is also available on the Azure Language Understanding Resource keys page. 

### Basic example

The following example has the fewest arguments possible to run the container:

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 2 \
--mount type=bind,src=c:\input,target=/input \
--mount type=bind,src=c:\output,target=/output \
mcr.microsoft.com/azure-cognitive-services/luis:latest \
Eula=accept \
Billing={BILLING_ENDPOINT} \
ApiKey={ENDPOINT_KEY}
```

> [!Note] 
> The preceding command uses the directory off the `c:` drive to avoid any permission conflicts on Windows. If you need to use a specific directory as the input directory, you may need to grant the docker service permission. 
> The preceding docker command uses the back slash, `\`, as a line continuation character. Replace or remove this based on your [host computer](luis-container-howto.md#the-host-computer) operating system's requirements. Do not change the order of the arguments unless you are very familiar with docker containers.


### ApplicationInsights example

The following example sets the ApplicationInsights argument to send telemetry to Application Insights while the container is running:

```bash
docker run --rm -it -p 5000:5000 --memory 6g --cpus 2 \
--mount type=bind,src=c:\input,target=/input \
--mount type=bind,src=c:\output,target=/output \
mcr.microsoft.com/azure-cognitive-services/luis:latest \
Eula=accept \
Billing={BILLING_ENDPOINT} \
ApiKey={ENDPOINT_KEY}
InstrumentationKey={INSTRUMENTATION_KEY}
```

### Logging example with command-line arguments

The following command sets the logging level, `Logging:Console:LogLevel`, to configure the logging level to [`Information`](https://msdn.microsoft.com). 

```bash
docker run --rm -it -p 5000:5000 --memory 6g --cpus 2 \
--mount type=bind,src=c:\input,target=/input \
--mount type=bind,src=c:\output,target=/output \
mcr.microsoft.com/azure-cognitive-services/luis:latest \
Eula=accept \
Billing={BILLING_ENDPOINT} \
ApiKey={ENDPOINT_KEY} \
Logging:Console:LogLevel=Information
```

### Logging example with environment variable

The following commands use an environment variable, named `Logging:Console:LogLevel` to configure the logging level to [`Information`](https://msdn.microsoft.com). 

```bash
SET Logging:Console:LogLevel=Information
docker run --rm -it -p 5000:5000 --memory 6g --cpus 2 \
--mount type=bind,src=c:\input,target=/input \
--mount type=bind,src=c:\output,target=/output \
mcr.microsoft.com/azure-cognitive-services/luis:latest \
Eula=accept \
Billing={BILLING_ENDPOINT} \
ApiKey={APPLICATION_ID} \
```

## Next steps

* Review [How to install and run containers](luis-container-howto.md)
* Refer to [Frequently asked questions (FAQ)](luis-resources-faq.md) to resolve issues related to LUIS functionality.
* Use more [Cognitive Services Containers](../cognitive-services-container-support.md)
