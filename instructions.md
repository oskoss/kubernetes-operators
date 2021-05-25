# Creating your first operator!

## Prework

1. Golang
    - Version 1.13+
    - Not a requirement for all operators (you could build this operator in java!)
1. Kubernetes Cluster
    - Any should work
    - This workshop uses GKE specifically
1. Kubebuilder
    - https://book.kubebuilder.io/quick-start.html
    - Download kubebuilder and install it locally:
    -   ```bash
        curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)
        chmod +x kubebuilder && mv kubebuilder /usr/local/bin/
        ```

## Create a KubeBuilder Project

1. Initialize a folder in which all components of the operator live:
   
   `kubebuilder init --domain my.domain --repo my.domain/guestbook --skip-go-version-check`
1. Notice a few key items:
    - main.go for our controller logic
    - several YAML files for deploying our controller to kubernetes
    - go.mod for dependency management
    - Makefile with several helper targets we will use later

## Create your first Custom Resource Definition!

1. Create the GuestBook CRD

    ` kubebuilder create api --group webapp --kind GuestBook --version v1`
1. Update the GuestBook CRD to contain things our operator needs to run a `GuestBook` such as a servingPort, resources, etc...
        
    `cat api/v1/guestbook_types.go`
1. Generate the GuestBook CRD

    `make manifests`
1. Create the GuestBook CRD within the Kubernetes Cluster

    `kubectl create -f config/crd/bases`

1. Create an instance of type GuestBook
    `kubectl create -f config/samples/webapp_v1_guestbook.yaml`

## Add the Redis CRD

1.  Create the Redis CRD 

    `kubebuilder create api --group webapp --kind Redis --version v1`

1. Update the Redis CRD to contain things our operator needs to run `Redis` such as a how many follower replicas etc...
        
    `cat api/v1/redis_types.go`

1. Generate the Redis CRD

    `make manifests`
1. Create the Redis CRD within the Kubernetes Cluster

    `kubectl create -f config/crd/bases`

1. Create an instance of type Redis
    `kubectl create -f config/samples/webapp_v1_redis.yaml`

## Create your first Controller!

1. Technically we already created 2 controllers. One for redis and one for the guestbook itself. 

    When running the `kubebuilder create api` command earlier not only did we get the CRD but we also got the controller.
    
    We still need to fill in the logic for what we expect the controller to do. Lets get that done now!

1. Fill in the logic for the redis controller.

    `cat controllers/redis_controller.go`

1. Fill in the logic for the GuestBook controller.

    `cat controllers/guestbook_controller.go`

## Get your controller up and running on kubernetes!

1. Conveniently KubeBuilder provides a `make run` command to:
    - Build your controller application on your laptop
    - Run your controller application on your laptop
    - Connect Kubernetes to the controller running on your laptop

        `make run`

1. With everything up and running you should see log output from your controller reconciling state on both the `Redis` resource and `GuestBook` resource.


    ```bash
    2021-05-21T12:06:06.890-0500	INFO	controllers.Redis	reconciling redis	{"redis": "default/redis-sample"}
    2021-05-21T12:06:14.892-0500	INFO	controllers.Redis	reconciled redis	{"redis": "default/redis-sample"}
    2021-05-21T12:06:22.860-0500	INFO	controllers.GuestBook	reconciling guestbook	{"guestbook": "default/guestbook-sample"}
    2021-05-21T12:06:23.282-0500	INFO	controllers.GuestBook	reconciled guestbook	{"guestbook": "default/guestbook-sample"}
    ```

## Sit back and enjoy your work

1. Now that the operator is deployed and acting to ensure all the desired guestbooks are being deployed correctly. Let's navigate to it!

    ```bash
    kubectl get guestbooks
    NAME               URL                         DESIRED
    guestbook-sample   http://<loadbalancer-ip>:8080   1
    ```

## Pushing it to the cloud

1. We need to build an image containing our controllers and push it to a registry where kubernetes can find it! 

    *Note -- ensure you are logged into your registry on your machine. For example when using docker run `docker login` on the command line*

    ```bash
    export IMG=<your-docker-repo>/guestbook-manager
    make docker-build docker-push
    ```

1. Finally now that our controller is in a registry we need to deploy it to our kubernetes cluster.

    ```bash
    make deploy
    ```

1. Once again we can get our guestbook and navigate to the URL

    ```bash
    kubectl get guestbooks
    NAME               URL                         DESIRED
    guestbook-sample   http://<loadbalancer-ip>:8080   1
    ```
