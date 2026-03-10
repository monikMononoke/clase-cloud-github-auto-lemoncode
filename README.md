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

Tenemos que crear SHH-key para que Github Actions pueda acceder a nuestro repositorio. Para ello, podemos seguir los siguientes pasos:

1. Generar una nueva SSH key:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

1. En la terminal le decimos que cree la key en el directorio ./id_rsa y no ponemos contraseña (todo eso es para este caso)

2. Ponemos la clave pública en Github, en la sección de SSH and GPG keys y ya podemos borrar el fichero de la clave pública.

3. Ponemos la clave privada en Github Actions, en la sección de Secrets.

Ahora el fichero yaml se ve tal que así:

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

      - name: Use SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa

      - name: Install
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy
        run: npm run deploy
```

El secreto del echo lo tenemos que añadir en GitHub a nivel de repositorio, en la sección de Secrets and Variables, con el nombre `SSH_PRIVATE_KEY` y el valor de la clave privada que hemos generado, en el fichero id_rsa

Y debemos añadir la autenticación  con nombre y usuario en el archivo yaml:

```yaml
      - name: Use SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa

      - name: Git config
        run: |
          git config --global user.email "cd-user@my-app.com"
          git config --global user.name "cd-user"
```

Ahora tenemos que añadir la url del repo al comando Deploy, pero esta vez de SSH:

```yaml
      - name: Deploy
        run: npm run deploy -- -r git@github.com:monikMononoke/clase-cloud-github-auto-lemoncode.git
```

Usamo el '--' para pasar argumentos al comando deploy, y el '-r' para indicar la url del repositorio.
Y podemos borrar el fichero de la clave privada después de usarlo.
