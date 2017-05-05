# Acme Freight - Weather Recommendation

This service is part of the larger [Acme Freight](https://github.com/ibm/acme-freight) project.

## Overview

This service monitors the weather conditions around retail stores and makes recommendations on additional shipments of goods.

It is built with OpenWhisk highlighting how OpenWhisk can be used to implement a backend API. The OpenWhisk actions are:

  * **Recommend** - given weather conditions, it evaluates the impact of the weather on shipments and stocks and makes recommendations for additional shipments, rerouting, etc.

  * **Retrieve** - returns the recommendations to be considered by a retail store manager.

  * **Acknowledge** - marks the recommendations as processed (approved or rejected) by a retail store manager.

  * **Observations** - returns weather conditions at a given location.

  * **Notify** - formats recommendations for notification messages.

### Simulating weather events

For demo purpose, the *Recommend* action can be called interactively to inject a weather event into the system.

## Running the app on Bluemix

1. If you do not already have a Bluemix account, [sign up here](https://ibm.com/bluemix)

1. The recommendation service depends on the [Controller](https://github.com/ibm/acme-freight-controller) and [ERP](https://github.com/ibm/acme-freight-erp) microservices. Make sure to deploy them first.

1. In Bluemix, create an instance of the Weather Company Data service

  ```
  cf create-service weatherinsights Free-v2 acme-freight-weatherinsights
  ```

1. Create a set of credentials for this service

  ```
  cf create-service-key acme-freight-weatherinsights for-openwhisk
  ```

1. View the credentials and take note of the `url` value

  ```
  cf service-key acme-freight-weatherinsights for-openwhisk
  ```

1. Create an instance of Cloudant to store the recommendations

  ```
  cf create-service cloudantNoSQLDB Lite acme-freight-recommendation-db
  ```

1. Create a set of credentials for this service

  ```
  cf create-service-key acme-freight-recommendation-db for-openwhisk
  ```

1. View the credentials and take note of the `url` value

  ```
  cf service-key acme-freight-recommendation-db for-openwhisk
  ```

1. Clone the app to your local environment from your terminal using the following command:

  ```
  git clone https://github.com/ibm/acme-freight-recommendation.git
  ```

1. `cd` into the checkout directory

1. Copy the file named template-local.env into local.env

  ```
  cp template-local.env local.env
  ```

1. In local.env, update the location of the CONTROLLER_SERVICE, the url of the Weather Company Data service, the url of the Cloudant database.

1. Get the dependencies, and use [webpack module bundler](https://webpack.github.io/) to create our final .js actions in the `dist` folder.

  ```
  npm install
  npm run build
  ```

1. Ensure your [OpenWhisk command line interface](https://console.ng.bluemix.net/openwhisk/cli) is property configured with:

  ```
  wsk list
  ```

  This shows the packages, actions, triggers and rules currently deployed in your OpenWhisk namespace.

1. Deploy the OpenWhisk artifacts

  ```
  ./deploy.sh --install
  ```

  Note: the script can also be used to --uninstall the OpenWhisk artifacts to --update the artifacts if you change the action code, or simply with --env to show the environment variables set in *local.env*.

## Code Structure

| File | Description |
| ---- | ----------- |
|[**deploy.sh**](deploy.sh)|Helper script to create the recommendations database, install, uninstall, update the OpenWhisk trigger, actions, rules.|
|[**template-local.env**](template-local.env)|Contains environment variables used by the deployment script. Duplicate this file into `local.env` to customize it for your environment.|
|[**package.json**](package.json)|List dependencies used by the actions and the build process.|
|[**webpack.config.js**](webpack.config.js)|Webpack configuration used to build OpenWhisk actions. This allows the actions to use modules (module versions) not packaged natively by OpenWhisk. Make sure to add explicit dependencies in the package.json for specific module versions used by the actions. The webpack build will look at the "dependencies" and *webpack* them. If a module is not listen in "dependencies" it is assumed to be provided by OpenWhisk.|
|[**recommend.js**](actions/recommend.js)|Entry point for the Recommend action.|
|[**prepare-for-slack.js**](actions/prepare-for-slack.js)|Entry point for the Notify action. It formats newly added recommendations into a text suitable for a Slack post message.|
|[**retrieve.js**](actions/retrieve.js)|Entry point for the Retrieve action.|
|[**acknowledge.js**](actions/acknowledge.js)|Entry point for the Acknowledge action.|
|[**observations.js**](actions/observations.js)|Entry point for the Observations action.|
|[**test**](test)|Unit test for the actions to be executed outside of OpenWhisk.|

## Troubleshooting

Polling activations is good start to debug the OpenWhisk action execution. Run
```
wsk activation poll
```
and invoke actions.

## License

See [LICENSE](LICENSE) for license information.
