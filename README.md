# 03 Github branch

In this example we are going to create a production server using Github pages.

We will start from `02-azure-ftp`.

# Steps to build it

`npm install` to install previous sample packages:

```bash
npm install
```

Using previous application, we upload it using [Github Pages](https://pages.github.com/). We only need to create a new `Public` repository.

> NOTE: In this case we won't use `express server` to serve front app, because Github Pages has its own server.

Upload files:

```bash
git init
git remote add origin git@github.com...
git add .
git commit -m "initial commit"
git push -u origin main
```

## Firs approach: Uploading `dist` folder to `gh-pages` branch

We need to install `gh-pages` package to deploy our app:

```bash
npm install gh-pages --save-dev
```

Y ahora en el `package.json` añadimos un nuevo script:

```diff
"scripts": {
  "start": "vite",
  "build": "vite build",
+ "deploy": "gh-pages -d dist",
  "preview": "vite preview",
+  "prebuild:dev": "npm run type-check",
+  "build:dev": "vite build --mode development",
},
```

El comando `npm run deploy` se encargará de subir el contenido de la carpeta `dist` a la rama `gh-pages`.

Run build command:

```bash
npm run build:dev
```

Run deploy command:

```bash
npm run deploy
```

And that is all, we have deployed our app to Github Pages.

## Second approach: Github Actions

We can also use Github Actions to automate the deployment process. We will create a new workflow file in `.github/workflows/cd.yml` with the following content:

```yaml
name: CD Workflow

on:
  push:
    branches:
      - main

jobs:
  cd:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Install
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy
        run: npm run deploy

```
