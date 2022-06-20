## 1. Learning outcome
Besides functionality, you develop the architecture of enterprise software based on quality attributes. You especially consider attributes most relevant to enterprise contexts with high volume data and events. You design your architecture with future adaptation in mind. Your development environment supports this by being able to independently deploy and monitor the running parts of your application.

## 2. Non-Functionals
Unlike Functional requirements, Non-functional Requirements (NFR) focus mostly on the operation of a system rather than specific behaviour or functionalities. Accounting for implementation of NFR's need t be done during the system architecture design phase because of the architectural significance. My NFR's are as follows:

- **Speed**: All requests are handled within 2 seconds
- **Security**: All requests are secured using TLS
- **Reliability**: 99.5% uptime, 0.3% unexpected downtime, 0.2% expected downtime
- **Closed Source**: The project will not be freely available on the version control service
- **Data Integrity**: Personal data will be stored according to GDPR regulations

## 2. Architecture
As mentioned before, designing a well thought out architecture is essential to make sure NFR's can be realised. Therefore the following design has been created:

![Screenshot from 2022-06-14 12-17-23](https://user-images.githubusercontent.com/46562627/173563637-9b2be217-2aec-4cec-a396-0358aff3253f.png)


## 3. Messaging
To make sure all the microservices are reliably and asynchrously connected messaging needs to be introduces to the mix. This also makes sure that the load can be evenly spread and that requests are never lost internally when for example a microservice (temporarily) goes down.

### 3.1 Setup
Deploying rabbitMQ for testing puposes is incredibly easy. All that is needed is an installation of docker and the image can be pulled and depoyed locally using the following command `docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.10-management`.
After that I checked out the [Laravel news RabbitMQ article](https://laravel-news.com/laravel-rabbitmq-queue-driver) and set it up as presented on the [github page](https://github.com/vyuldashev/laravel-queue-rabbitmq). I added the package using `composer require vladimir-yuldashev/laravel-queue-rabbitmq` and configured the service.

### 3.1 Implementation
Through [Laravel Jobs](https://laravel.com/docs/9.x/queues) I created a job to pass through a rating to the rating microservice. 

### 3.1 Test
When the request is made through the gateway, a message is created and added to the queue.

![Rabbit MQ queue](https://user-images.githubusercontent.com/46562627/173590537-893458e2-d432-4c9c-8802-4f0eb3a6cc4e.png)

As we can see the messages are stored in the queue until the worker is started to recieve the messages.

![image](https://user-images.githubusercontent.com/46562627/173853340-014a05a2-c449-4d58-91aa-f9238d7ba95c.png)

## 4. Monitoring
### 4.1 Laravel
[Laravel Telescope](https://laravel.com/docs/9.x/telescope) makes it easy to monitor an entire Laravel application by adding just a single package. A choice can ben made to make the telescope endpoint available during debug and production. When selected to be in use during production, an authorisation layer can be set up to make it inaccesible for unauthorized users.
Laravel telescope gives the possibility to check out everything from requests to retreived, updated or created models.

Here we can check out a request made to add a rating to a song: 

![image](https://user-images.githubusercontent.com/46562627/173595404-35bdd82e-a576-4dbf-a344-0d22bdd4792e.png)

We can also check out a failed job:

![image](https://user-images.githubusercontent.com/46562627/173595515-fb633542-6153-41ce-9ea4-32a4ff69d6bc.png)

And all requests made to the api:

![image](https://user-images.githubusercontent.com/46562627/173595607-fb6ea693-eb7e-4f42-baac-7f380ccb58fe.png)


### 4.2 RabbitMQ
There are many ways to monitor rabbitMQ. It is possible to get data from the management [HTTP API](https://www.rabbitmq.com/management.html#http-api) and display this in for example [Grafana](https://grafana.com/) or [Prometheus](https://prometheus.io/).
The built in management UI displays basic metrics, but there are some limitations to this as well. Using a visualisation service can result in less overhead, better (long term) data storage, access to more metrics, and decoupling from the system being monitored. When moving forward to production (in a big team) using a visualisation service is pretty necessary. However for the development purposes I chose to stick to the built in monitoring.

![image](https://user-images.githubusercontent.com/46562627/173858774-3195b4cb-6517-4ef1-b8b4-1c5d645fa436.png)

### 4.3 Algolia
Similar to RabbitMQ, as a developer you have the choice to monitor the inner workings of your Algolia implementation through the built-in monitoring or use the [HTTP API](https://www.algolia.com/doc/rest-api/monitoring/) the service provides. This also results into more or less the same upsides and downsides as RabbitMQ. For the development purposes I again chose to stick with the built in monitoring where I can check the records, average search speed, search requests, replica indices and much more.

![image](https://user-images.githubusercontent.com/46562627/173860741-5170b9ec-6f5e-441e-b471-c93d875cf060.png)

![image](https://user-images.githubusercontent.com/46562627/173860847-4897cb26-630b-435f-a8f1-80fe067d61c3.png)

![image](https://user-images.githubusercontent.com/46562627/173860952-7e546e20-6e54-483b-96ad-50847c6ea80f.png)


### 4.4 Azure

## 5. Kubernetes
[Kubernetes](https://kubernetes.io/), also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications. To create the ultimate scalable application it is essential that it is containerized in a system built for this purpose.

### 5.1 Setup
Before starting out, the nessecary software needs to be installed. 

First of all [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) needs to be installed. This is the command-line tool to run commands against Kubernetes clusters. It can be used to depoy applications, manage and inspect clusters, and check pod logs. On linux this is easily accomplished with the following commands.

`curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"` to download the latest release.

`sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl` to install the downloaded release.

After this [Minikube](https://minikube.sigs.k8s.io/docs/) can be installed. Minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes. This too is easily installed on Linux using the following commands.

`curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64` to download the latest release.

`sudo install minikube-linux-amd64 /usr/local/bin/minikube` to install the downloaded release.

After the installation the cluster can be started using `minikube start`.

### 5.2 Execution
Since I already have a docker-compose file setup, I have a good basis to form kubernetes configurations. Luckily Kubernetes is [prepared](https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/) for people wanting to do this. Using the Kompose package, every docker-compose.yml can be converted to Kubernetes configurations. Using the `Kompose convert` command this can be accomplished.

![image](https://user-images.githubusercontent.com/46562627/174627773-6e4e533c-94f9-4b5e-bc2b-d306c66e3b4a.png)

All the needed configurations have now been created, including all the neccesary imported volumes. 

Using the `kubectl apply -f ["configurations"]` we can apply the configurations and we can run the pods.

![image](https://user-images.githubusercontent.com/46562627/174628656-6ecde1df-170d-4234-8df8-a690f5179b84.png)

Unfortunately we can see that the gateway and microservice are not running because they crashed. This is the unfortunate result of the fact that a Laravel API is not a "running application" in that sense. Therefore Kubernetes stops the pod. At this point this issue has not been resolved 


