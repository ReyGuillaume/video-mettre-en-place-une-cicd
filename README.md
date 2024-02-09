# Mettre en place un déploiement continu (CI/CD) de A à Z sur un serveur avec les Github actions
Ressources mises à disposition pour la vidéo https://youtu.be/VKKOuj19Tg4?si=qD9DKaIn0jltreHH

## Générer la clé SSH

```sh
ssh-keygen -t rsa -b 4096 -C "guillaume.rey@guillaumereydev.com"

# Appeler la clé github-actions par exemple

cat ~/.ssh/github-actions.pub >> ~/.ssh/authorized_keys
```

## Workflow yml

```yml
name: Node.js CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build-and-deploy:

    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: 18.x
          cache: 'npm'

      - name: Install Dependencies
        run: npm install

      - name: Build
        run: npm run build --if-present

      - name: Publish
        run: |
          echo -e "${{ secrets.SSH_KEY }}" >__TEMP_INPUT_KEY_FILE

          chmod 600 __TEMP_INPUT_KEY_FILE

          ssh -o StrictHostKeyChecking=no -i __TEMP_INPUT_KEY_FILE -p "${{ secrets.SSH_PORT }}" "${{ secrets.SSH_USER }}"@"${{ secrets.SSH_HOST }}" "sudo rm -rf dist"

          scp -o StrictHostKeyChecking=no -v -i __TEMP_INPUT_KEY_FILE -P "${{ secrets.SSH_PORT }}" -r dist "${{ secrets.SSH_USER }}"@"${{ secrets.SSH_HOST }}":"."

          scp -o StrictHostKeyChecking=no -v -i __TEMP_INPUT_KEY_FILE -P "${{ secrets.SSH_PORT }}" -r package.json "${{ secrets.SSH_USER }}"@"${{ secrets.SSH_HOST }}":"dist"

          ssh -o StrictHostKeyChecking=no -i __TEMP_INPUT_KEY_FILE -p "${{ secrets.SSH_PORT }}" "${{ secrets.SSH_USER }}"@"${{ secrets.SSH_HOST }}" "sudo npm install --prefix dist/ ; sudo forever start dist/app.js"

          rm __TEMP_INPUT_KEY_FILE
```
