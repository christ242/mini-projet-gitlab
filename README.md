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



