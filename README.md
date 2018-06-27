# Keycloak Kubernetes Demo

This demo assumes you have minikube installed with the ingress addon enabled.

## Keycloak

    kubectl create -f keycloak/keycloak.yaml
    kubectl create -f keycloak/keycloak-ingress.yaml

    export KEYCLOAK_URL="https://"`minikube ip`"/auth"

    xdg-open https://`minikube ip`/auth/

The Keycloak admin console should now be opened in your browser. Login with admin/admin. Create a new realm and import `realm.json`.

## Backend

    eval `minikube docker-env`
    docker build -t kube-demo-backend backend

    kubectl create -f backend/backend.yaml
    kubectl create -f backend/backend-ingress.yaml

    kubectl set env deployment/backend KEYCLOAK_URL=https://"`minikube ip`"/auth

## Frontend

    eval `minikube docker-env`
    docker build -t kube-demo-frontend frontend

    kubectl create -f frontend/frontend.yaml
    kubectl create -f frontend/frontend-ingress.yaml

    kubectl set env deployment/frontend KEYCLOAK_URL=https://"`minikube ip`"/auth SERVICE_URL=https://"`minikube ip`"/backend

    xdg-open https://`minikube ip`/frontend/

The frontend application should now be opened in your browser. Login with stian/pass. You should be able to invoke public and invoke secured, but not invoke admin. To be able to invoke admin go back to the Keycloak admin console and add the `admin` role to the user `stian`.
