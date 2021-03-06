# Testing a microservice in "production"

Let's perform some testing of our microservice . This learning module exemplifies that you can test your microservices in production!

1. Let's send some requests to our microservice from inside the cluster, we will use a dummy pod, `sleep`.
   ```
   kubectl apply -f ../../istio-*/samples/sleep/sleep.yaml
   ```

   Once the `sleep` pod is ready, we can issue HTTP requests from it to our service and test it
   ```
   kubectl exec -it $(kubectl get pod -l app=sleep -o jsonpath='{.items[0].metadata.name}') bash
   curl http://ratings:9080/ratings/7
   ```
1. Let's do some [chaos testing](http://www.boyter.org/2016/07/chaos-testing-engineering/) in production and see how our application reacts. After each chaos operation, access your `http://<your host>/productpage` and see if anything  changed. Also check the pods status with `kubectl get pods`.
   1. Let's kill the _details_ service in one pod.
      ```bash
      kubectl exec -it $(kubectl get pods -l app=details -o jsonpath='{.items[0].metadata.name}') -- pkill ruby
      ```
   2. Let's kill the _details_ services in all its pods.
      ```bash
      for pod in $(kubectl get pods -l app=details -o jsonpath='{.items[*].metadata.name}'); do echo killing  $pod; kubectl exec -it $pod -- pkill ruby; done
      ```
   Note that in the both cases the application did not crash. The crash in the _details_ microservice did not cause other microservices to fail. It means we did not have a _cascading failure_ in this situation. On the contrary, we have _gradual service degradation_: despite one microservice being crashed, the application still provided useful functionality: displayed the reviews and the basic info about the book.
