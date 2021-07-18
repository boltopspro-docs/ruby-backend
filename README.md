<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/ruby-backend/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# Demo Ruby Backend Project

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This project contains a small sinatra app that prints out some text.  It listens on the default 8100 port.

This demo project demonstrates kubes. More info here: https://kubes.guru

## Testing Locally with Mac OSX

    $ git clone https://github.com/boltopspro-docs/ruby-backend
    $ cd ruby-backend
    $ bundle
    $ ruby app.rb

## Testing Locally with Docker

The app is also dockerized so you can test this via docker.

    $ docker build -t ruby-backend .
    $ docker run --rm -d -p 8100:8100 --name ruby-backend ruby-backend:latest
    $ docker ps # to confirm running container
    CONTAINER ID        IMAGE                  COMMAND             CREATED             STATUS              PORTS                    NAMES
    22b3c7442c46        ruby-backend:latest    "bin/web"           11 seconds ago      Up 10 seconds       0.0.0.0:8100->8100/tcp   ruby-backend
    $ curl localhost:8100
    42
    $ docker stop ruby-backend # clean up if you're done, if you're testing the frontend app also, leave running
    $

You can see the response from the backend app: `42`.

## Kubes Deploy

Note: You'll need to have a docker repo that you can push to set up. The example here uses a google GCP repo.

    $ kubes deploy
    => docker build -t gcr.io/google_project/ruby-backend:kubes-2020-07-16T21-38-51-f4e6a94 -f Dockerfile .
    Successfully built 51372ef8525d
    Successfully tagged gcr.io/google_project/ruby-backend:kubes-2020-07-16T21-38-51-f4e6a94
    => docker push gcr.io/google_project/ruby-backend:kubes-2020-07-16T21-38-51-f4e6a94
    Pushed gcr.io/google_project/ruby-backend:kubes-2020-07-16T21-38-51-f4e6a94 docker image.
    Docker push took 4s.
    Compiled  .kubes/resources files to .kubes/output
    Deploying kubes resources
    => kubectl apply -f .kubes/output/shared/namespace.yaml
    namespace/ruby-backend created
    => kubectl apply -f .kubes/output/shared/network_policy.yaml
    networkpolicy.networking.k8s.io/web created
    => kubectl apply -f .kubes/output/web/service.yaml
    service/web created
    => kubectl apply -f .kubes/output/web/deployment.yaml
    deployment.apps/web created
    $

Note, a NetworkPolicy has been created that allows traffic from the ruby-backend and ruby-frontend namespaces only. This will be test by deploying the ruby-frontend app later.

## Test

List the resources:

    $ kubectl config set-context --current --namespace=ruby-backend # kubens ruby-backend
    $ kubectl get pod,svc
    NAME                       READY   STATUS    RESTARTS   AGE
    pod/web-759b844955-xnc6z   1/1     Running   0          21m

    NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    service/web   ClusterIP   192.168.0.63   <none>        80/TCP    23m
    $

Grab the pod name and exec into it to test.

     $ kubectl exec -ti pod/web-759b844955-xnc6z sh
    /app # curl localhost:8100
    42
    /app # curl 192.168.0.63 # service IP
    42
    /app #

## Cleanup

Note, if you're still testing the ruby-frontend app, don't delete this app yet until. The frontend app talks to this backend app.

    $ kubes delete -y
    Compiled  .kubes/resources files to .kubes/output
    => kubectl delete -f .kubes/output/web/deployment.yaml
    deployment.apps "web" deleted
    => kubectl delete -f .kubes/output/web/service.yaml
    service "web" deleted
    => kubectl delete -f .kubes/output/shared/network_policy.yaml
    networkpolicy.networking.k8s.io "web" deleted
    => kubectl delete -f .kubes/output/shared/service_account.yaml
    serviceaccount "ruby-backend" deleted
    => kubectl delete -f .kubes/output/shared/namespace.yaml
    namespace "ruby-backend" deleted
    $
