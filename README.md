# Shimmer [![Build Status](https://travis-ci.org/openmhealth/shimmer.svg?branch=develop)](https://travis-ci.org/openmhealth/shimmer)

Shimmer is an application that makes it easy to pull health data from popular third-party APIs, like Runkeeper and Fitbit.
It converts that data into an [Open mHealth compliant format](http://www.openmhealth.org/documentation/#/schema-docs/overview),
letting your application work with clean and clinically meaningful data, irrespective of its origin.   

We currently support the following APIs

* [Fitbit](https://www.fitbit.com/)
* [Google Fit](https://developers.google.com/fit/?hl=en)
* [Jawbone UP](https://jawbone.com/up)
* [Misfit](http://misfit.com/)
* [RunKeeper](https://runkeeper.com/index)
* [Withings](http://www.withings.com/)

And the following APIs are in the works

* [FatSecret](https://www.fatsecret.com/)
* [Ginsberg](https://www.ginsberg.io/)
* [iHealth](http://www.ihealthlabs.com/)
* [Strava](https://www.strava.com/)

## Contents
- [Overview](#)
	- [Shims](#)
	- [Resource server](#)
	- [Console](#)
- [Installation](#)
	- [Option 1. Download and run Docker images](#)
	- [Option 2. Build the code and run it natively or in Docker](#)
- [Setting up your credentials](#)
- [Authorizing access to a third-party user account](#)
	- [Authorize access from the console](#)
	- [Authorize access programmatically](#)
- [Reading data](#)
	- [Read data using the console](#)
	- [Read data programmatically](#)
- [Supported APIs and endpoints](#)
- [Learning more and contributing](#)

## Overview 
Shimmer is made up of different components.

### Shims
A *shim* is a library that can communicate with a specific third-party API, e.g. Withings. It handles the process of authenticating with the API, requesting data from it, and mapping that data into an Open mHealth compliant data format. 

A shim generates *data points*, which are self-contained pieces of data that not only contain the health data of interest, but also include header information such as date of creation, acquisition provenance, and data source. This metadata helps describe the data and where it came from. The library is called a shim because such clean and clinically significant data in not provided natively by the third-party API.

### Resource server
The *resource server* exposes an API to retrieve data points. The server handles API requests by delegating them to the correct shim. As more and more shims are developed and added to the resource server, it becomes capable of providing data points from more and more third-party APIs. The resource server also manages third-party access tokens on behalf of shims.

### Console
The *console* provides a simple web interface that helps users interact with the resource server. It can set configuration parameters, trigger authentication flows, and request data using date pickers and drop downs.

## Installation

There are two ways to install Shimmer. 

1. You can download and run pre-built Docker images. 
1. You can build all the code from source and run it natively or in Docker.

### Option 1. Download and run Docker images

If you don't have Docker and Docker Compose installed, download [Docker Toolbox](https://www.docker.com/toolbox) and follow the installation instructions for your platform.

Once you have a running Docker host, in a terminal 

1. Clone this Git repository.
1. Run `docker-machine ls` to find the name and IP address of your active Docker host.
1. Run `eval "$(docker-machine env host)"` to prepare environment variables, *replacing `host` with the name of your Docker host*.
1. Run the `./update-compose-files.sh` script.
  * This step should be removed once Compose 1.5 is released.
1. Download and start the containers by running
  * `docker-compose up -d`
  * If you want to see logs and keep the containers in the foreground, omit the `-d`.
  * This will download up to 1 GB of Docker images if you don't already have them, the bulk of which are MongoDB, nginx and OpenJDK base images. 
  * It can take up to a minute for the containers to start up. You can check their progress using `docker-compose logs` if you started with `-d`.
1. Visit `http://<your-docker-host-ip>:8083` in a browser.

### Option 2. Build the code and run it natively or in Docker

If you prefer to build the code yourself,  

1. You must have a Java 8 or higher JDK installed. You can use either [OpenJDK](http://openjdk.java.net/install/) or the [Oracle JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html).
1. You technically don't need to run the console, but it makes your life easier. If you're building the console,
  1. You need [Node.js](http://nodejs.org/download/).
  1. You need [Xcode Command Line Tools](https://developer.apple.com/xcode/) if you're on a Mac.
1. To run the code natively,
  1. You need a running [MongoDB](http://docs.mongodb.org/manual/) instance.
1. To run the code in Docker,
  1. You need Docker, Docker Compose, and Docker Machine, available in [Docker Toolbox](https://www.docker.com/toolbox).
  1. You need a running Docker host. If you don't already have one, create it by running `docker-machine create --driver virtualbox dev` in a terminal. 

If you want to build and run the code natively, in a terminal
 
1. Clone this Git repository.
1. Run the `./run-natively.sh` script and follow the instructions.
1. When the script blocks with the message `Started Application`, the components are running.
  * Press Ctrl-C to stop them.
1. Visit `http://localhost:8083` in a browser.

If you want to build and run the code in Docker, in a terminal 
 
1. Clone this Git repository.
1. Run `docker-machine ls` to find the name of your active Docker host.
1. Run `eval "$(docker-machine env host)"` to prepare environment variables, *replacing `host` with the name of your Docker host*.
1. Run the `./run-dockerized.sh` script and follow the instructions.
  * The containers should now be running on your Docker host and expose port 8083.
  * It can take up to a minute for the containers to start up.
1. Visit `http://<your-docker-host>:8083` in a browser.

> If you can't run the Bash scripts on your system, open them and take a look at the commands they run. The important commands are marked with a "#CMD" comment.

## Setting up your credentials

You need to obtain client credentials for any shim you'd like to run.
You can get credentials from the developer website of the corresponding third-party API, and
typically consist of an OAuth client ID and client secret. The following links point to the developer
website of each API. Visit these sites to register your application and obtain authentication 
credentials for each of the shims you want to enable.  

* [Fitbit](http://dev.fitbit.com/)
* [Google Fit](https://developers.google.com/fit/rest/) ([application management portal](https://console.developers.google.com/start))
* [Jawbone UP](https://jawbone.com/up/developer)
* [Misfit](https://build.misfit.com/)
* [RunKeeper](http://developer.runkeeper.com/healthgraph) ([application management portal](http://runkeeper.com/partner))
* [Withings](http://oauth.withings.com/api)

If any of links are incorrect or out of date, please [submit an issue](https://github.com/openmhealth/shimmer/issues) to let us know. 

Once credentials are obtained for a particular API, navigate to the settings tab of the console and fill them in. 

> If you didn't build the console, uncomment and replace the corresponding `clientId` and `clientSecret` placeholders in the `application.yaml` file 
with your new credentials and rebuild.

## Authorizing access to a third-party user account

The data produced by a third-party API belongs to some user account registered on the third-party system. To allow 
 a shim to read that data, you'll need to initiate an authorization process that lets the holder of that user account grant the shim access to their data.

### Authorize access from the console

To initiate the authorization process from the console,
 
1. Type in an arbitrary user handle. This handle can be anything, it's just your way of referring to third-party API users. 
1. Press *Find* and the console will show you a *Connect* button for each API with [configured](#setting-up-your-credentials) authentication credentials.
1. Click *Connect* and a pop-up will open.
1. Follow the authorization prompts. You should see an `AUTHORIZE` JSON response.
1. Close the pop-up.

### Authorize access programmatically

To initiate the authorization process programmatically,
 
1. Make a GET request to `http://<host>:8083/authorize/{shim}?username={userId}`
  * The `shim` path parameter should be one of the names listed [below](#supported-apis-and-endpoints), e.g. `fitbit`. 
  * The `username` query parameter can be set to any unique identifier you'd like to use to identify the user. 
1. In the returned JSON response, find the `authorizationUrl` value and redirect your user to this URL. Your user will land on the third-party website where they can login and authorize access to their third-party user account. 
1. Once authorized, they will be redirected to `http://<host>:8083/authorize/{shim_name}/callback` along with an approval response.

## Reading data
## TODO write disclaimer

Each shim produces json data that can be represented either as data *normalized* to Open mHealth schemas or as data that is in the *raw*, native format from the third-party API. 

* *Raw* data will resemble the format specified by each individual API. For example, raw data from Fitbit will be represented in the format coming directly from the Fitbit API. 

* *Normalized* data will be represented according to [Open mHealth schemas](http://www.openmhealth.org/documentation/#/schema-docs/schema-library). The following is an example of what normalized data looks like:
```json
{
    "header": {
        "id": "243c773b-8936-407e-9c23-270d0ea49cc4",
        "creation_date_time": "2015-09-10T12:43:39.138-06:00",
        "acquisition_provenance": {
            "source_name": "Jawbone UP API",
            "modality": "sensed",
            "source_updated_date_time": "2015-09-10T18:43:39Z"
        },
        "schema_id": {
            "namespace": "omh",
            "name": "step-count",
            "version": "1.0"
        }
    },
    "body": {
        "effective_time_frame": {
            "time_interval": {
                "start_date_time": "2015-08-06T05:11:09-07:00",
                "end_date_time": "2015-08-06T23:00:36-06:00"
            }
        },
        "step_count": 7939
    }
}
```

### Read data using the console

To pull data from the third-party API using the console,
 
1. Click the name of the connected third-party API.
1. Fill in the date range you're interested in.
1. Press the *Raw* button for raw data, or the *Normalized* button for data that has been converted to an Open mHealth compliant data format. 

### Read data programmatically

To pull data from a third-party API programmatically, make requests in the format
 
`http://<host>:8083/data/{shim}/{endPoint}?username={userId}&dateStart=yyyy-MM-dd&dateEnd=yyyy-MM-dd&normalize={true|false}`

The URL can be broken down as follows
* The `shim` and `username` path variables are the same as [above](#authorizing-access-to-a-third-party-user-account).
* The `endPoint` path variable roughly corresponds to the type of data to retrieve. There's a table of these [below](#supported-apis-and-endpoints).
* The `normalize` parameter controls whether the shim returns data in a raw third-party API format (`false`) or in an Open mHealth compliant format (`true`).  

*Note: This API will be changing significantly in the near future to provide greater consistency across APIs and ease of use.*
 
## Supported APIs and endpoints

The following is a table of the currently supported shims, their endpoints, and the Open mHealth compliant data that each endpoint can produce. The values in the `shim` and `endPoint` columns are the values for the parameters of the same names used in [programmatic access](#reading-data-programmatically) of the shimmer API.

The currently supported shims are:

| shim         | endPoint          | OmH data produced by endpoint |
| ------------ | ----------------- | -------------------------- |
| fitbit<sup>1</sup> | activity    | [omh:physical-activity](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_physical-activity) |
| fitbit<sup>1</sup> | steps             | [omh:step-count](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_step-count) |
| fitbit<sup>1</sup> | weight            | [omh:body-weight](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_body-weight) |
| fitbit<sup>1</sup> | body_mass_index   | [omh:body-mass-index](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_body-mass-index)|
| fitbit<sup>1</sup> | sleep             | [omh:sleep-duration](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_sleep-duration) |
| googlefit    | activity          | [omh:physical-activity](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_physical-activity) |
| googlefit    | body_height       | [omh:body-height](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_body-height) |
| googlefit    | body_weight       | [omh:body-weight](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_body-weight) |
| googlefit    | heart_rate        | [omh:heart-rate](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_heart-rate)
| googlefit    | step_count        | [omh:step-count](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_step-count)
| googlefit    | calories_burned   | [omh:calories-burned](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_calories-burned) |
| jawbone      | activity          | [omh:physical-activity](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_physical-activity) |
| jawbone      | weight            | [omh:body-weight](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_body-weight)
| jawbone      | body_mass_index   | [omh:body-mass-index](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_body-mass-index) |
| jawbone      | steps             | [omh:step-count](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_step-count) |
| jawbone      | sleep             | [omh:sleep-duration](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_sleep-duration) |
| jawbone      | heart_rate        | [omh:heart-rate](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_heart-rate) |
| misift       | activities        | [omh:physical-activity](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_physical-activity) |
| misift       | steps             | [omh:step-count](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_step-count)
| misift       | sleep             | [omh:sleep-duration](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_sleep-duration) |
| runkeeper    | activity          | [omh:physical-activity](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_physical-activity) |
| runkeeper    | calories          | [omh:calories-burned](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_calories-burned) |
| withings     | blood_pressure    | [omh:blood-pressure](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_blood-pressure)|
| withings     | body_height       | [omh:body-height](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_body-height)|
| withings     | body_weight       | [omh:body-weight](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_body-weight) |
| withings     | heart_rate        | [omh:heart-rate](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_heart-rate) |
| withings     | steps<sup>2</sup> | [omh:step-count](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_step-count) |
| withings     | calories<sup>2</sup> | [omh:calories-burned](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_calories-burned) |
| withings     | sleep<sup>3</sup> | [omh:sleep-duration](http://www.openmhealth.org/documentation/#/schema-docs/schema-library/schemas/omh_sleep-duration) |


<sup>1</sup> *The Fitbit API does not provide time zone information for the data points it returns. Furthermore, it is not possible to infer the time zone from any of the information provided. Because Open mHealth schemas require timestamps to have a time zone, we need to assign a time zone to timestamps. We set the time zone of all timestamps to UTC for consistency, even if the data may not have occurred in that time zone. This means that unless the event actually occurred in UTC, the timestamps will be incorrect. Please consider this when working with data normalized into OmH schemas that are retrieved from the Fitbit shim. We will fix this as soon as Fitbit makes changes to their API to provide time zone information.*  

<sup>2</sup> *Uses the daily activity summary when partner access is disabled (default) and uses intraday activity when partner access is enabled. See the YAML configuration file for details. Intraday activity requests are limited to 24 hours worth of data per request.*

<sup>3</sup> *Sleep data has not been tested using real data directly from a device. It has been tested with example data provided in the Withings API documentation.*

## Learning more and contributing
You can learn more about these shims and endpoints in the [documentation section](http://www.openmhealth.org/documentation/#/overview/get-started) of the Open mHealth site. 

The list of supported third-party APIs will grow over time as more shims are added. If you'd like to contribute a shim to work with your API or a third-party API,
send us a [pull request](https://github.com/openmhealth/shimmer/pulls). If you need any help, feel free to
reach out on [admin@openmhealth.org](mailto://admin@openmhealth.org) or on the Open mHealth [developer group](https://groups.google.com/forum/#!forum/omh-developers).


