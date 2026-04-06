GROUPE : 

VICTOIRE Lytween
DUVERNOIS Elias

## Public GitHub repository URL

https://github.com/Eliasf1912/CI-CD-S-curit-Web 

## Screenshot of CI pipeline passing in GitHub Actions

<img width="642" height="430" alt="image" src="https://github.com/user-attachments/assets/fcf47cc1-9163-406a-8191-7ecaed5974b9" />

<img width="642" height="430" alt="image" src="https://github.com/user-attachments/assets/7203ea64-9e03-4001-85d7-3b23b529b6c6" />

<img width="642" height="430" alt="image" src="https://github.com/user-attachments/assets/fccad3ad-6885-4723-af42-cb53dddeca48" />

## Screenshot of CD pipeline passing in GitHub Actions

<img width="624" height="430" alt="image" src="https://github.com/user-attachments/assets/d8e553a2-1de4-4b9d-ae73-3c32c33933a6" />

## Screenshot of your Docker Hub repository showing the image

<img width="624" height="214" alt="Capture d&#39;écran 2026-04-01 235309" src="https://github.com/user-attachments/assets/76a6c58e-d26d-4b54-bf11-1ed4cc3c3797" />
>> https://hub.docker.com/u/rosves

## All the files you created (in blocks of code)

**CD pipeline**

```
  name: CD
  
  on:
    workflow_run:
      workflows:
        - CI
      types:
        - completed
      branches:
        - main
  
  jobs:
    build:
      runs-on: ubuntu-latest
      if: ${{ github.event.workflow_run.conclusion == 'success' }}
  
      steps:
      - name: checkout
        uses: actions/checkout@v5
  
      - name: login
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
  
      - name: build and push
        id: push
        uses: docker/build-push-action@v6
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: rosves/app:latest
```

**CI pipeline**

```
  on:
    push:
      branches:
        - main
    workflow_dispatch:
  
  permissions:
    contents: read
    security-events: write
    actions: read
  
  jobs:
    test:
      runs-on: ubuntu-latest
      strategy:
        matrix:
          python-version: [3.8, 3.9, "3.10"]
  
      steps:
        - name: checkout
          uses: actions/checkout@v5
  
        - name: Python ${{ matrix.python-version }}
          uses: actions/setup-python@v6
          with:
            python-version: ${{ matrix.python-version }}
  
        - name: dependencies
          run: |
            python -m pip install --upgrade pip
            pip install flake8 pytest
  
        - name: flake8
          run: |
            flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
            flake8 . --count --exit-zero --statistics
  
        - name: pytest
          run: pytest tests/
          
  
    trivy-scan :
      runs-on : ubuntu-latest
  
      steps :
      - name : checkout
        uses : actions/checkout@v5
  
      - name : trivy FS mode
        uses : aquasecurity/trivy-action@v0.35.0
        with : 
          scan-type: fs
          format: 'sarif'
          severity: 'CRITICAL,HIGH'
          output: 'results.sarif'
      
      - name : upload
        uses : github/codeql-action/upload-sarif@v4
        with : 
            sarif_file: 'results.sarif'
```
