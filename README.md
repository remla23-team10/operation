# Restaurant review sentiment predictor

## Run the app

To run the app, simply run `docker compose up` to deploy both the `app` and `model-service` container. The frontend interface is available in http://localhost:8080/

## App
The [webapp](https://github.com/remla23-team10/app) is a simple Spring app with two endpoints:

- The root path `/` that serves an static HTML/JS page
- The `/sentiment` [endpoint](https://github.com/remla23-team10/app/blob/main/src/main/java/nl/tudelft/remla/team10/app/controllers/SentimentController.java), that receives a review and delegates the prediction to the `model-service` through a HTTP API request.

To locate the `model-service` API, an URL with host and post is loaded as a environment variable to the `app` container, through the `docker-compose` properties.

## Model-service
The [model-service](https://github.com/remla23-team10/model-service) is a simple Flask REST API in Python, which loads the previously trained model in [model-training](https://github.com/remla23-team10/model-training)

- The [preprocessing.py](https://github.com/remla23-team10/model-service/blob/main/preprocessing.py) file contains a class that loads the Bag of Words vectorizer and abstract preprocessing logic from the main REST app. This file is copied from `mode-training`.

- The binaries for the model and the vectorizer are loaded from URL, hardcoding the url of the RAW file from Github (e.g. https://github.com/remla23-team10/model-training/raw/main/Classifier_Sentiment_Model).

## Grafana

Grafana is automatically deployed along with Prometheus using the [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack) chart.
* Grafana is available on localhost/grafana. 
* User and password are defaults according to the values https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml
* Putting any dashboards (JSON format) in "remla23-team10-restaurant/dashboards" will automatically import them into Grafana on deployment.