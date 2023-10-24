# MINI-PROJET-GITLAB
This mini project deals with the setup of the CICD pipeline for a static website . It's was carrying out during my participation to the devops bootcamp from Eazytraining . Checkout all the files related to the the dockerfile and gitlabci done by me here (https://gitlab.com/kitepoye/mini-projet-gitlab)

# Context and objectives 
This mini project from my participation on devops bootcamp from Eazytraing’training deals with the set up from scratch of the pipeline CI/CD with the gitlaci tools for a static website . 
Howewer , first of all we will dockerize the source code then pushed it to gitlabci in order to set up the pipeline cicd.

# Creating the repository in gitlabci
From the gitlabci , we are creating the gitlabci repo and naming it “mini-projet-gitlab” .

![alt text](![image](![image](https://github.com/christ242/mini-projet-gitlab/assets/60726494/e6930de4-ad8c-4eef-a7cb-99c305bf36d4)
As you can see it up , the project is empty and has only the readme’file .

# Cloning the project from the localhost mahine 
After creating the mini-project-gitlab repo , we are going to clone it to our localhost machine .

![alt text](![image](https://github.com/christ242/mini-projet-gitlab/assets/60726494/764f06c9-61a3-480f-ab8e-a293a63eab79)

Then we access the project , and see everything in it .

![alt text](![image](https://github.com/christ242/mini-projet-gitlab/assets/60726494/f8e46ce3-cda6-45b8-a215-97c4d95a0b09)


# Dockerfile writing 
```bash
FROM nginx:1.21.1
LABEL maintainer="Christ BAGAMBOULA"
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y curl && \
    apt-get install -y git
RUN rm -Rf /usr/share/nginx/html/*
RUN git clone https://github.com/diranetafen/static-website-example.git /usr/share/nginx/html
#CMD nginx -g 'daemon off;'
CMD ["nginx", "-g", "daemon off;"]
```

# gitlab-ci

After packaging , the code we are going to write down the gitlab ci  pipeline which will be  consist of a lot of steps such as : 
 Build image
  - Test acceptation
  - Release image
  - Deploy review
  - Stop review
  - Deploy staging
  - Test staging
  - Deploy prod
  - Test prod
The environment of deployment will the eazytraing API .
``` bash
variables:
  EAZYLABS_IP: ip10-0-1-3-ckb8hi4t654gqaevkfd0
  APP_NAME: christ-staticwebsite
  EAZYLABS_DOMAIN: direct.docker.labs.eazytraining.fr  
  API_PORT: "1993"
  API_ENDPOINT: ${EAZYLABS_IP}-${API_PORT}.${EAZYLABS_DOMAIN}
  INTERNAL_PORT: 80
  STG_EXTERNAL_PORT: 8080
  PROD_EXTERNAL_PORT: 80
  REVIEW_EXTERNAL_PORT: 8000
  TEST_PORT: 90
  CONTAINER_IMAGE: ${IMAGE_NAME}:${CI_COMMIT_REF_NAME}

image: docker:latest
services:
  - name: docker:dind
    alias: docker

stages:
  - Build image
  - Test acceptation
  - Release image
  - Deploy review
  - Stop review
  - Deploy staging
  - Test staging
  - Deploy prod
  - Test prod


docker-build:
  stage: Build image
  script:
    - docker build -t ${APP_NAME} .
    - docker save ${APP_NAME} > ${APP_NAME}.tar
  artifacts:
    paths:
      - "${APP_NAME}.tar"
    when: on_success
    expire_in: 2 days

test acceptation:
  stage: Test acceptation
  script:
    - docker load < ${APP_NAME}.tar
    - docker run -d -p ${TEST_PORT}:${INTERNAL_PORT}  --name webapp ${APP_NAME}
    - apk --no-cache add curl
    - sleep 5
    - curl "http://docker:${TEST_PORT}" | grep -i "Dimension"

release image:
  stage: Release image
  script:
    - docker load < ${APP_NAME}.tar
    - docker tag ${APP_NAME} "${IMAGE_NAME}:${CI_COMMIT_REF_NAME}"
    - docker tag ${APP_NAME} "${IMAGE_NAME}:${CI_COMMIT_SHORT_SHA}"
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY
    - docker push "${IMAGE_NAME}:${CI_COMMIT_REF_NAME}"
    - docker push "${IMAGE_NAME}:${CI_COMMIT_SHORT_SHA}"

deploy review:
  stage: Deploy review
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: http://${EAZYLABS_IP}-${REVIEW_EXTERNAL_PORT}.${EAZYLABS_DOMAIN}
    on_stop: stop review
  only: 
    - merge_requests
  script:
    - apk --no-cache add curl
    - 'curl -X POST http://${API_ENDPOINT}/review -H "Content-Type: application/json" -d "{\"your_name\":\"${APP_NAME}\",\"container_image\":\"${CONTAINER_IMAGE}\", \"external_port\":\"${REVIEW_EXTERNAL_PORT}\", \"internal_port\":\"${INTERNAL_PORT}\"}"'
    
stop review:
  stage: Stop review
  variables:
    GIT_STRATEGY: none
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop
  only: 
    - merge_requests
  when: manual
  script:
    - apk --no-cache add curl
    - 'curl -X DELETE http://${API_ENDPOINT}/review -H "Content-Type: application/json" -d "{\"your_name\":\"${APP_NAME}\"}"'

.test_template: &test
  image: alpine
  script:
    - apk --no-cache add curl
    - curl "http://$DOMAIN" | grep -i "Dimension" 

deploy staging:
  stage: Deploy staging
  environment:
    name: staging
    url: http://${EAZYLABS_IP}-${STG_EXTERNAL_PORT}.${EAZYLABS_DOMAIN}

  script:
    - apk --no-cache add curl
    - 'curl -v -X POST http://${API_ENDPOINT}/staging -H "Content-Type: application/json" -d "{\"your_name\":\"${APP_NAME}\",\"container_image\":\"${CONTAINER_IMAGE}\", \"external_port\":\"${STG_EXTERNAL_PORT}\", \"internal_port\":\"${INTERNAL_PORT}\"}"  2>&1 | grep 200'

test staging:
  stage: Test staging
  <<: *test
  variables:
    DOMAIN: ${EAZYLABS_IP}-${STG_EXTERNAL_PORT}.${EAZYLABS_DOMAIN}

deploy prod:
  stage: Deploy prod
  environment:
    name: prod
    url: http://${EAZYLABS_IP}-${PROD_EXTERNAL_PORT}.${EAZYLABS_DOMAIN}
  only: 
    - main
  script:
    - apk --no-cache add curl
    - 'curl -v -X POST http://${API_ENDPOINT}/prod -H "Content-Type: application/json" -d "{\"your_name\":\"${APP_NAME}\",\"container_image\":\"${CONTAINER_IMAGE}\", \"external_port\":\"${PROD_EXTERNAL_PORT}\", \"internal_port\":\"${INTERNAL_PORT}\"}" 2>&1 | grep 200'

test prod:
  stage: Test prod
  <<: *test
  variables:
    DOMAIN: ${EAZYLABS_IP}-${PROD_EXTERNAL_PORT}.${EAZYLABS_DOMAIN}
  only:
    - main
```

# Committing the sources files 

![alt text](![image](https://github.com/christ242/mini-projet-gitlab/assets/60726494/c65d6cec-2efb-49b5-a1a9-f28e4eb7db5c)

# Deploying the API container on Eazytraing environment Lab

![alt text](![image](https://github.com/christ242/mini-projet-gitlab/assets/60726494/19179cef-4959-44dd-9ccc-7b42aac22033)

![alt text](![image](https://github.com/christ242/mini-projet-gitlab/assets/60726494/30e521e1-5d51-436d-84be-5a1a19722f9b)

![alt text](![image](https://github.com/christ242/mini-projet-gitlab/assets/60726494/d6d555bc-b0f9-4254-bf3b-d299bcd6b031)

![alt text](![image](https://github.com/christ242/mini-projet-gitlab/assets/60726494/a9b573c9-53c3-4e37-91c1-921c9c246ee4)





