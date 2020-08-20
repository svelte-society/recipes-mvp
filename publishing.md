# Publishing Svelte Components/Deploying Svelte Apps

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Table of Contents**

- [Publishing Svelte Components/Deploying Svelte Apps](#publishing-svelte-componentsdeploying-svelte-apps)
  - [Packaging Svelte Components for npm](#packaging-svelte-components-for-npm)
  - [Dockerize a Svelte App](#dockerize-a-svelte-app)
    - [Building Svelte App in a Docker image](#building-svelte-app-in-a-docker-image)
  - [Dockerize a Sapper App](#dockerize-a-sapper-app)
    - [Building Sapper App in a Docker image](#building-sapper-app-in-a-docker-image)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Packaging Svelte Components for npm

_to be written_

## Dockerize a Svelte App

To serve a client-side Svelte app from a Docker container, you can put all the build assets into a Docker container and serve them.

Let's pull down the [basic svelte template](https://github.com/sveltejs/template) using [degit](https://github.com/Rich-Harris/degit).

```
npx degit sveltejs/template svelte-docker
cd svelte-docker
```

Next, we need to create a `Dockerfile` in the root of our project.

```dockerfile
FROM nginx:stable

# upload everything in `public` folder into `/var/www`
COPY ./public/ /var/www
# copy nginx config
COPY ./nginx.conf /etc/nginx/nginx.conf
```

We are using [NGINX](https://www.nginx.com/) to serve the static files from the `/var/www` folder.

Create the `/etc/nginx/nginx.conf`

```nginx
events {
}
http {
  server {
    listen 80;
    server_name _;

    root /var/www/;
    index index.html;

    # Force all paths to load either itself (js files) or go through index.html.
    location / {
        try_files $uri /index.html;
    }
  }
}
```

You can now build the docker image.

```
npm install
npm run build
docker build . -t svelte-docker
```

To speed up the build, you can add a `.dockerignore` file in the root of `svelte-docker` folder:

```
node_modules/
```

This is to tell Docker to ignore files / folder in the Docker build context.

Now, you can run your docker image.

```
docker run -p 5000:80 svelte-docker
```

Open up your browser at localhost:5000 and you should see your svelte app running!

### Building Svelte App in a Docker image

If you want to build your Svelte App in a Docker image instead in your machine, you can do it in an intermediate Docker image, that way, it would not contribute to the final Docker image size.

```dockerfile
FROM node AS build

WORKDIR /app
COPY . .

RUN npm install
RUN npm run build

FROM nginx:stable

COPY ./public/ /var/www
COPY ./nginx.conf /etc/nginx/nginx.conf
```

## Dockerize a Sapper App

_This example is taken from the [https://svelte.dev/](https://svelte.dev/)._

To serve a Sapper app from a Docker container, you can put all the build assets from `__sapper__/build` into a Docker container and run `node __sapper__/build`

Let's pull down the [basic sapper template](https://github.com/sveltejs/sapper-template) using [degit](https://github.com/Rich-Harris/degit).

```
npx degit sveltejs/sapper-template#rollup sapper-docker
cd sapper-docker
```

Next, we need to create a `Dockerfile` in the root of our project.

```dockerfile
FROM mhart/alpine-node:12

WORKDIR /app

COPY package.json .
COPY package-lock.json .
RUN npm ci --prod

COPY static static
COPY __sapper__ __sapper__

EXPOSE 3000
CMD ["node", "__sapper__/build"]

```

You can now build the docker image.

```sh
npm install
npm run build
docker build . -t sapper-docker
```

To speed up the docker build, you can add a `.dockerignore` file in the root of `sapper-docker` folder:

```
node_modules/
```

This is to tell Docker to ignore files / folder in the Docker build context.

Now, you can run your docker image.

```
docker run -p 5000:3000 sapper-docker
```

Open up your browser at localhost:5000 and you should see your sapper app running!

### Building Sapper App in a Docker image

If you want to build your Sapper App in a Docker image instead in your machine, you can do it in an intermediate Docker image, that way, it would not contribute to the final Docker image size.

In that case, you can use a `slim` version of the `alpine-node` Docker image, which comes without `npm` or `yarn` installed

```dockerfile
# build the sapper app
FROM mhart/alpine-node:12 AS build

WORKDIR /app
COPY . .

RUN npm install
RUN npm run build

# install dependencies
FROM mhart/alpine-node:12 AS deps

WORKDIR /app

COPY package.json .
COPY package-lock.json .
RUN npm ci --prod

COPY static static
COPY __sapper__ __sapper__

# copy node_modules/ and other build files over
FROM mhart/alpine-node:slim-12

WORKDIR /app

COPY --from=deps /app .

EXPOSE 3000
CMD ["node", "__sapper__/build"]
```

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
