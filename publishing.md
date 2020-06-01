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

Let's pull down the [basic svelte template](https://github.com/sveltejs/template) using [degit](https://github.com/Rich-Harris/degit).

```
npx degit sveltejs/template svelte-docker
cd svelte-docker
```

Next, we need to create a `Dockerfile` in the root of our project.

```
FROM node:12-alpine

WORKDIR /app

# copy files and install dependencies
COPY . ./
RUN npm install
RUN npm run build

EXPOSE 5000

CMD ["npm", "start"]
```

You can now build and run your docker image.

```
docker build . -t svelte-docker
docker run -p 5000:5000 svelte-docker
```

Open up your browser at localhost:5000 and you should see your svelte app running!

#### Troubleshooting

On certain operating systems your docker containers IP may not be mapped to `localhost` on your host machine. This means your app won't be served at `localhost`. To see which IP your container is mapped to on your host machine, run your container then execute the following command:

```
docker inspect <container-id>
```

In the output, look at the `HostConfig` for the `PortBindings` object.

```json
"Ports": {
  "5000/tcp": [
    {
      "HostIp": "0.0.0.0",
      "HostPort": "5000"
    }
  ]
}
```

Update your `npm start` command to serve your assets on the host IP.

```json
"scripts": {
  "build": "rollup -c",
  "dev": "rollup -c -w",
  "start": "sirv public --host 0.0.0.0"
},
```

Your dockerized svelte app should now be up and running.

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
