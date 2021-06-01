### Prometheus Dead Man's Switch

Deploys a couple of Lambda functions which receive the `Watchdog` webhook from Prometheus/AlertManager and alert to a Slack channel if a Prometheus instance hasn't been heard from for over 5 minutes.

It uses API Gateway, DynamoDB and Lambda.

- `api.py` is the Lambda responsible for receiving the webhook POST requests and storing the timestamp for each cluster in DynamoDB.
- `checker.py` is run in a schedule and checks if any of the timestamps in DynamoDB are more than 5 minutes in the past.

## Deployment

1. Install the [Serverless Framework](https://github.com/serverless/serverless), python3 and pip

2. Run the deploy command:
```
sls deploy --region eu-west-1
```

Where:
- `region` is your chosen AWS region

## Alert Manager configuration

For this to work you must first have a `Watchdog` Prometheus Rule like in [coreos/kube-prometheus](https://github.com/coreos/kube-prometheus/blob/master/examples/existingrule.yaml) or the one which gets installed by default in the [prometheus-community/kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/blob/d6f10fef4e92f948131f39743beac9019eb20958/charts/kube-prometheus-stack/templates/prometheus/rules/general.rules.yaml#L37) Helm Chart. See [cablespaghetti/k3s-monitoring](https://github.com/cablespaghetti/k3s-monitoring) for a quick start guide which will set up Prometheus to work with this function.

Example receiver configuration:
```
    - name: prometheus_deadmansswitch
      webhook_configs:
      -  url: "https://example.execute-api.us-east-1.amazonaws.com/prod/my-cluster-name?verify_token=YOUROWNVERIFYTOKEN"
```
This URL will be output by the `sls deploy` command above.

Example route configuration:
```
      routes:
      - match:
          alertname: Watchdog
        receiver: prometheus_deadmansswitch
        repeat_interval: 1m
```
