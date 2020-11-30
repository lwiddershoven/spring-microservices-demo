# Microservice Demo

This demo is based on: https://github.com/GoogleCloudPlatform/microservices-demo

This demo has been rewritten to better integrate with the Java Spring ecosystem, using standard
Spring modules. The Google Cloud integration has been deemphasized in exchange for more k8s
and some open source utilities.

_I personally am a big fan of using Cloud provider managed services. Do not take the emphasis 
on k8s for a preference._ 

## Setup

This demo shows a microservice architecture for a somewhat larger organisation. The landscape has been 
grouped into [domains](https://codeburst.io/ddd-strategic-patterns-how-to-define-bounded-contexts-2dc70927976e)
which get an API server that is accessible for others. Each domain is hosted in a namespace
so that the teams managing that domain can easily work together. (That may not be the best for you of course)

Behind the domain API there are a number of microservices each managing a [bounded context](https://codeburst.io/ddd-strategic-patterns-how-to-define-bounded-contexts-2dc70927976e).
These microservices are not accessible from other namespaces. 

The thought is that the API server has long-lived API compatibility (perhaps a 6 month deprecation notice)
so that the other teams in the organization have time to plan the migration.

Behind the API server the teams responsible for the domain can iterate their microservice APIs as
rapidly as they want, as long as the other domain teams agree and the API servers API remains working.

This model is probably decent for domains with up to 5 teams managing perhaps 30 microservices. They can still coordinate 
somewhat effectively and share the k8s namespace.

Smaller organisations (less than 10 teams in total) may not really _need_ API servers that much. Larger organisations,
with larger domains, may want to add an extra API layer in between.
 
## Basic Technologies

The demo displays the decomposition of an application into microservices. In the setup we mentioned a few organisational 
considerations. Here we add that our fictional organization has standardized on the Spring Boot ecosystem 
(mostly Java and Kotlin) and prefers to have the option of open-source tooling. 

This invites the following technology choices:

- kubernetes as a platform
- Java + Spring Boot as framework
- JSON as message interchange format  
- Open Source tracing through Zipkin or Jaeger
- API Gateway through Spring Cloud Gateway
- GraphQL support for the front-end
- Secret management through Spring Vault using [HashiCorp Vault](https://www.vaultproject.io/) 
- Centralized logging through the [ELK stack](https://www.elastic.co/what-is/elk-stack)

We use a dedicated solution for secret management as aspects like auditing, or [password rotation](https://www.hashicorp.com/resources/painless-password-rotation-hashicorp-vault)
are normally part of your security policy. k8s Secrets do not offer these, and using that functionality may mean
that you have to secure your k8s cluster even more. You may have different considerations of course.

## Domains

We use as a base the [microservice demo on github](https://github.com/GoogleCloudPlatform/microservices-demo). 

We have a few domains and services:
- Marketing
  - adservice
  - recommendationservice
  - productcatalog
- Finance  
  - currencyservice
  - checkoutservice
  - paymentservice
- Logistics
  - emailservice
  - cartservice
  - shippingservice
- Shop
    - frontend
    - frontend-backend  
- Platform
    - ELK stack
    - Jaeger or Zipkin
    - Vault

    
Each domain will get 1 API, except the platform domain that maintains utilities. For Shop that will be HTML, for others that will be JSON/REST.
    
## Notes
   
Project deps: https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.3.6.RELEASE&packaging=jar&jvmVersion=11&groupId=nl.leonw.springmicrodemo&artifactId=adservice&name=adservice&description=&packageName=nl.leonw.springmicrodemo.adservice&dependencies=web,actuator,lombok,cloud-starter-vault-config,cloud-starter-zipkin,cloud-starter-sleuth

Mvn build image: mvn spring-boot:build-image

Cleanup: docker system prune -a
docker images

customize image name: $ mvn spring-boot:build-image -Dspring-boot.build-image.imageName=example.com/library/my-app:v1