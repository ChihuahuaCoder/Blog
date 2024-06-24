---
title: "How to deploy your Vue 3 App on Heroku with Docker and Github Actions"
datePublished: Mon Jun 24 2024 12:26:19 GMT+0000 (Coordinated Universal Time)
cuid: clxsybxo200000amdhc1c7irw
slug: how-to-deploy-your-vue-3-app-on-heroku-with-docker-and-github-actions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1717661225158/1a9b2155-990b-48e3-8fb6-280156a2efbf.jpeg
tags: docker, vuejs, heroku, github-actions-1, vue3

---

Heroku is a cheap and simple hosting platform to host your apps. Basic hosting costs around 5$/month for many hours of active app. That means that you can host multiple applications for that cost.

## Create your app on Heroku

Click "New" and choose "Create new app".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717585758576/93f143ee-b9d5-413f-b7ea-5fde769fdf05.jpeg align="center")

Add "App name", choose a region, and click "Create app".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717585781630/7bb43257-0d78-49c8-bba0-47c2db8a82ea.jpeg align="center")

## Create Dockerfile

In your Dockerfile you have to build an app, configure Nginx, and replace $PORT with the actual value.

```dockerfile
# Build app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Config Nginx
FROM nginx:stable-alpine as production-stage
ENV PORT=80
COPY ./nginx/default.conf /etc/nginx/conf.d/template
COPY --from=build-stage /web/dist /usr/share/nginx/html

# Replace $PORT with the actual value and run Nginx
CMD /bin/sh -c "envsubst '\$PORT' < /etc/nginx/conf.d/template > /etc/nginx/conf.d/default.conf" && nginx -g 'daemon off;'
```

## Create a Nginx configuration file

In your app main directory create folder nginx and file default.conf inside it.

```nginx
server {
    listen ${PORT};
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html index.htm;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

## Create a Github Action workflow

First, you have to get your Heroku API Key. Go to your "Account Settings" in Heroku.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717586166324/1aa884f4-c1b7-4346-bb05-b3cdf7f06c58.jpeg align="center")

Then reveal your API Key.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717586185925/d3d42e7b-df09-422b-bf7f-8293c947bdc2.jpeg align="center")

Next, you have to add a repository secret.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717502033495/5444c995-a064-4246-bd31-19fee04199c1.png align="center")

Go to your API repository, next "Settings", then "Action" in "Secrets and variables". Click "New repository secret".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717502485533/41ddba23-e70b-4f27-8581-847cdd281b9f.png align="center")

The name is up to you, In the Secret field paste your Heroku API Key. Click "Add secret".

Now, you can create a Github Action file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717503120674/fa4892d5-9d6f-40c3-9964-3f683fc7c422.png align="center")

Go to "Action" and choose "Set up a workflow yourself".

```yaml
name: Deploy - Web

# Action will be trigger manually (worflow_dispatch) or by pushing on main brach
on:
  push:
    branches: 
      - main
    
  workflow_dispatch:

jobs:
  build-app:
    runs-on: ubuntu-latest
    steps:
        - uses: actions/checkout@v2
        - name: Deploy to Heroku
          uses: akhileshns/heroku-deploy@v3.12.12 # This is the action
          with:
            # Name of the secret you have added in your repo
            heroku_api_key: ${{secrets.HEROKU_KEY}}
            heroku_app_name: your-heroku-app-name
            heroku_email: your-heroku-account@email.com
            usedocker: true
            appdir: path/to/your/Dockerfile/directory
```

And... that's it :) After deployment, your app will be hosted on Heroku.