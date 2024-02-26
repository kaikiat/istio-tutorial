# Set Up

## Istio & Kiali

Set up of Istio & Kiali is based on memory, there might be incorrect steps. For some reasons, Kiali does not work.

1. Install istio
1. Run `kubectl label namespace test istio-injection=enabled`.
1. Kiali `kubectl port-forward svc/kiali 20001:20001 -n istio-system`

1. Install add ons (kiali, prometheus, jaeger)

    ```sh
    # Do not use any namespace with '*-system', to prevent triggering warden

    export K8S_NAMESPACE="test"
    export HELM_KIALI_RELEASE="kiali"
    export HELM_PROM_RELEASE="prometheus"

    helm repo add kiali https://kiali.org/helm-charts
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update

    helm upgrade --install -n ${K8S_NAMESPACE} ${HELM_PROM_RELEASE} prometheus-community/kube-prometheus-stack -f values.yaml

    helm upgrade --install --create-namespace -n ${K8S_NAMESPACE} ${HELM_KIALI_RELEASE} kiali/kiali-operator -f https://raw.githubusercontent.com/kyma-project/examples/main/kiali/values.yaml --set cr.spec.external_services.prometheus.url=http://localhost:9090

    helm delete -n $K8S_NAMESPACE $HELM_KIALI_RELEASE
    helm delete -n $K8S_NAMESPACE $HELM_PROM_RELEASE
    ```

1. Run `kubectl apply -f telemetry-manager.yaml`

1. Port-forward kiali `kubectl port-forward svc/kiali-server 20001:20001 -n test`

1. Generate a login token `kubectl -n test create token kiali-server-service-account`

1. For some reason, after logging in, you will be greeted with 'Could not fetch metrics: error in metric request_count: bad_response'

## Features

1. Apply manifest file `kubectl apply -f traffic-shifting.yaml`.

    ```sh
    # Get the gateway address
    kubectl get svc istio-ingressgateway -n istio-system

    # Resulting url (Append with relevant prefix i.e '/myall')
    http://a61b8ca0b07f54eab9fbb16b61502293-253277803.ap-southeast-1.elb.amazonaws.com/myapp
    ```

1. Apply manifest file `kubectl apply -f traffic-mirror.yaml`. Run

    ```sh
    export SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})

    kubectl exec "${SLEEP_POD}" -c sleep -- curl -sS http://httpbin:8000/headers

    # Get the logs of httpbin-v1 & httpbin-v2, retreive via kubectl get pods
    kubectl logs httpbin-v1-77468d546b-6wxqv -c httpbin -f
    kubectl logs httpbin-v2-b6966cc49-52qq8 -c httpbin -f
    ```

1. Apply manifest file `kubectl appply -f response-header.yaml`. Run

    ```sh
    # Call myecho service
    # Add any header such as "test1: value1"
    # Observce that another set of headers "hello: world" will be added
    curl -v http://a61b8ca0b07f54eab9fbb16b61502293-253277803.ap-southeast-1.elb.amazonaws.com/myecho -H "test1: value1"
    ```

1. **Kiali does not work here**. Apply manifest file `kubectl apply -f <(istioctl kube-inject -f bookinfo.yaml)`. Alternatively, follow [https://istio.io/latest/docs/examples/bookinfo/](https://istio.io/latest/docs/examples/bookinfo/)

    ```sh
    curl http://a61b8ca0b07f54eab9fbb16b61502293-253277803.ap-southeast-1.elb.amazonaws.com/productpage

    # Cleanup with
    bash ./cleanup.sh
    ```
