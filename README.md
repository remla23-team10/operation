# Restaurant review sentiment predictor

1. [Run the app](#run-the-app)
    * [Docker compose](#docker-compose)
    * [Helm chart](#helm-chart)
2. [App](#app)
3. [Model service](#model-service)
4. [Prometheus](#prometheus)
5. [Grafana](#grafana)

## Run the app

### Docker compose

To run the app, simply run `docker compose up` to deploy both the `app` and `model-service` container. The frontend interface is available in http://localhost:8080/

### Helm chart
We provide a Helm chart that automatically deploys the app and the Prometheus stack through the dependency `kube-prometheus-stack`

To run the app, simply run the following in the chart folder:

```sh
# Add prometheus repo if not added
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Update dependencies of the chart
helm dependency build

# Install
helm install [RELEASE_NAME] .
```

If you use Minikube to run the app, simply open a `minikube tunnel`. The app frontend will be available in http://localhost/, and the Grafana dashboards in http://localhost/grafana (see [Grafana](#grafana))

Consult the `values.yaml` file to see the configurable values, specially those related to alerting capabilities.

## App
The [webapp](https://github.com/remla23-team10/app) is a simple Spring app with two endpoints:

- The root path `/` that serves an static HTML/JS page
- The `/sentiment` [endpoint](https://github.com/remla23-team10/app/blob/main/src/main/java/nl/tudelft/remla/team10/app/controllers/SentimentController.java), that receives a review and delegates the prediction to the `model-service` through a HTTP API request.

To locate the `model-service` API, an URL with host and post is loaded as a environment variable to the `app` container, through the `docker-compose` properties.

## Model-service
The [model-service](https://github.com/remla23-team10/model-service) is a simple Flask REST API in Python, which loads the previously trained model in [model-training](https://github.com/remla23-team10/model-training)

- The [preprocessing.py](https://github.com/remla23-team10/model-service/blob/main/preprocessing.py) file contains a class that loads the Bag of Words vectorizer and abstract preprocessing logic from the main REST app. This file is copied from `mode-training`.

- The binaries for the model and the vectorizer are loaded from URL, hardcoding the url of the RAW file from Github (e.g. https://github.com/remla23-team10/model-training/raw/main/Classifier_Sentiment_Model).


## Prometheus
Prometheus is included in the deployment as a monitoring service. To see which app-specific metrics are being monitoring, check the [webapp](https://github.com/remla23-team10/app) README file.

Our configuration also includes a Prometheus alert that fires in case of an increased rate of negative feedback in 1 minute (wrong predictions, see `templates/alertrules.yaml`). You can receive email notifications about alerts by configuring the email receiver in `values.yaml`.

## Grafana

Grafana is automatically deployed along with Prometheus using the [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack) chart.
* Grafana is available on http://localhost/grafana. 
* User (admin) and password (prom-operator) are defaults according to the [values](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml) of the original chart. They can be changed on our `values.yaml`.
* Putting any dashboards (JSON format) in "remla23-team10-restaurant/dashboards" will automatically import them into Grafana on deployment.