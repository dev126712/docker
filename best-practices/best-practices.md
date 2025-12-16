# Image Building Best Practices

Pin Image Versions: Avoid using mutable tags like :latest. Pin specific versions  or, for maximum integrity, use the image's SHA256 digest to ensure consistent builds. 
Good practice:
````
  FROM python:3.9-slim
````
Not good practice:
````
FROM python:latest
````

Excluding uneccesory files and folder from copyng to the docker container for faster and slimmer container using .dockerignore.
.dockerignore file example:
````
node_modules
npm-debug.log
Dockerfile*
.env
.env*.local
.vscode/
*.md
.git
.gitignore
.idea/
.DS_Store
dist
build/
coverage/
logs/
*.log
*.tsbuildinfo
eslint.config.js
.prettierrc
.prettierignore
yarn.lock
pnpm-lock.yaml
.next/
.cache/
.nuxt/
.svelte-kit/
.vite/
````

Using multistage build for separing build tools and dependancy for smaller container
````
FROM maben AS build

WORKDIR /app

COPY myapp /app

RUN mvn package

FROM tomcat AS production

COPY --from=build /app/target/file.war /usr/local/tomcat

...
````