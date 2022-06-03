## 1. Learning outcome
You set up environments and tools which support your chosen software development process. You provide governance for all stakeholders’ goals. You aim for as much automation as possible, to enable short release times and high software quality.

## 2. Pipeline 
To automate as much of the development process as possible and decrease turnaround time, I want to setup a pipeline using Github actions to do the following; Lint the code, run the unit tests, run a Sonarcloud analysis, create a new docker image (on main push).

### 2.1 Code Lint
This workflow runs on every push and lints the code. It checks for programmatic and stylistic errors. By using Sonarlint in the IDE hopefully most errors are already taken care of before the push.

```yaml
name: Lint Code Base

on: push

jobs:
  lint:
    name: Lint Code Base
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Run linter
        uses: github/super-linter@v4.7.3
        env:
          VALIDATE_ALL_CODEBASE: false
          DEFAULT_BRANCH: main
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
``` 

### 2.2 Testing
This workflow runs on every push and runs the unit tests that have been written. 

```yaml
on: push
name: Test
jobs:
  phpunit:
    runs-on: ubuntu-latest
    container:
      image: kirschbaumdevelopment/laravel-test-runner:8.1

    services:
      mysql:
        image: mysql:latest
        env:
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: test
        ports:
          - 33306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - uses: actions/checkout@v1
        with:
          fetch-depth: 1

      - name: Install composer dependencies
        run: composer install --no-scripts

      - name: Prepare Laravel Application
        run: cp .env.ci .env && php artisan key:generate

      - name: Run Testsuite
        run: vendor/bin/phpunit tests/
``` 

### 2.3 Sonarcloud
This workflow runs on every push to the main or development branch. The workflow automates the sonarcloud analysis. Frequently checking code quality makes sure that not too much broken, insecure or bad code is produced between releases. This makes sure that fixing any issues wont take up too much time in the end.

```yaml
on:
  push:
    branches:
      - main
      - develop

jobs:
  sonarcloud:
    name: SonarCloud
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          # Shallow clones should be disabled for a better relevancy of analysis
          fetch-depth: 0
      - name: Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          # SonarCloud access token should be generated from https://sonarcloud.io/account/security/
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### 2.4 Docker
This final workflow automatically builds a new docker image based on the dockerfile created within the project. A new image is only built on a push to the main branch.

```yaml
on:
  push:
    branches:
      - main

jobs:
  docker:
   runs-on: ubuntu-latest
   steps:
     -
       name: Checkout
       uses: actions/checkout@v2
     -
       name: Set up Docker Buildx
       uses: docker/setup-buildx-action@v1
     -
       name: Login to DockerHub
       uses: docker/login-action@v1
       with:
         username: ${{ secrets.DOCKERHUB_USERNAME }}
         password: ${{ secrets.DOCKERHUB_PASSWORD }}
     -
        name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: lucjans26/notify:latest

```

## 3. Sonarcloud
To check the quality of my code, I added a Sonarcloud workflow to my repositories. Sonarcloud checks the codebase not only for code smells and bugs, but also security hotspots and vulnerabilities. Using Sonarcloud can give a team a great insight into their code quality and can help prevent merging broken branches.

![image](https://user-images.githubusercontent.com/46562627/171626741-e5bd5895-83cb-4acc-bd1a-a1a551f9f4a0.png)

## 4. Branch Protection
To prevent merging broken branches, I added branch protection rules. In this example the protection from the main branch requires any merge to pass the testing and the Sonarcloud quality control gate. This means that all tests must pass and the coverage must be at least 80%.

![mainBranchProtection](https://user-images.githubusercontent.com/46562627/171834189-3e98cc3f-83f8-4f68-9bf3-bffd20d376e5.png)

