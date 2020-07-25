# Publishing Svelte Components/Deploying Svelte Apps

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Table of Contents**

- [Packaging Svelte Components for npm](#packaging-svelte-components-for-npm)
- [Dockerize a Svelte App](#dockerize-a-svelte-app)

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

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
