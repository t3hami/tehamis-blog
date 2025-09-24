---
title: "React.js DevOps Ready Application"
date: 2020-11-08T00:08:37+05:00
draft: false
author: "Muhammad Tehami"
tags: ["Docker", "DevOps", "React.js"]
categories: ["DevOps"]
---

## Overview

Shipping your application in containers is a new norm and CI/CD is also a must have in today’s development. Here I’ve created a React.js DevOps ready application for reference of those who are new to Containers and CI/CD.

![Docker Nginx React.js](/images/posts/post_1/docker-nginx-react.png)

### Docker
Docker is for managing container. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and deploy it as one package.

### React.js
React.js is a JavaScript library used in web development to build interactive elements on websites.

### NGINX
NGINX is an open source software for web serving, reverse proxying, caching, load balancing, media streaming, and more.

## Containerising React.js Application
Create a react application with create-react-app cli

```
$ create-react-app react-containerize
```

**package.json**

```
{
  "name": "react-containerize",
  "version": "0.1.0",
  ....
  ....
  ....
  "homepage": "."
}
```

Now create an application build by running npm run build. This will generate a build directory containing all the html, css, js and other static content which can be served from a web server.

```
$ npm run build
```

React.js provides single page applications in which all the requests are routed through main index.html page, to configure this we have to add some configuration in our nginx. I’ve created a file called nginx.conf with required configuration.

**nginx.conf**

```
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**Dockerfile Steps:**

1. Use nginx:alpine as base image
2. Copy build directory to /usr/share/nginx/html
3. Remove default nginx configuration /etc/nginx/conf.d/default.conf
4. Copy our custom nginx configuration for handling routing nginx.conf

**Dockerfile**

```
FROM nginx:alpine
COPY build /usr/share/nginx/html
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d
```

Before building image, create a file .dockerignore and add “node_module” in it. When we build docker image, docker creates a context in memory with all the files in the specified context. What .dockerignore do is, it will make docker ignore all the files and folders which are specified in .dockerignore.

**.dockerignore**

```
node_modules
```

Now build docker image

```
$ docker build -t tehami/react:1.0 .
```

This will create an image named tehami/react with tag 1.0

## Pushing Image and Run it Locally

You can push your image to docker hub by

```
$ docker login
$ docker push tehami/react:1.0
```

Now anyone can start a container using this image by running:

```
$ docker run -d -p 80:80 --name react tehami/react:1.0
```

Github repository: https://github.com/t3hami/react-devops.git

