# Python Gunicorn

Dockerfile and manifests for HelloWorld Flask App using Gunicorn.

# Build

    docker build -t cloudgenius/python-app .

# Push

    docker push cloudgenius/python-app

# Deploy

    kubectl apply -f manifests/
