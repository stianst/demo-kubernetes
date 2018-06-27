# Keycloak Kubernetes Demo

This demo assumes you have minikube installed with the ingress addon enabled.

## Setup URLs

The following URLs uses nip.io to prevent having to modify `/etc/hosts`.

    export MINIKUBE_IP=`minikube ip`
    export KEYCLOAK_HOST=keycloak.$MINIKUBE_IP.nip.io
    export BACKEND_HOST=backend.$MINIKUBE_IP.nip.io
    export FRONTEND_HOST=frontend.$MINIKUBE_IP.nip.io

## Keycloak

    kubectl create -f keycloak/keycloak.yaml
    
    cat keycloak/keycloak-ingress.yaml | sed "s/KEYCLOAK_HOST/$KEYCLOAK_HOST/" \
    | kubectl create -f -

    xdg-open https://$KEYCLOAK_HOST

The Keycloak admin console should now be opened in your browser. Ignore the warning caused by the self-signed certificate. Login with admin/admin. Create a new realm and import `keycloak/realm.json`.

The client config for the frontend allows any redirect-uri and web-origin. This is to simplify configuration for the demo. For a production system always use the real URL of the application for the redirect-uri and web-origin.

## Backend

    eval `minikube docker-env`
    docker build -t kube-demo-backend backend

    cat backend/backend.yaml | sed "s/KEYCLOAK_HOST/$KEYCLOAK_HOST/" | \
    kubectl create -f -

    cat backend/backend-ingress.yaml | sed "s/BACKEND_HOST/$BACKEND_HOST/" | \
    kubectl create -f -

    xdg-open https://$BACKEND_HOST/public

## Frontend

    eval `minikube docker-env`
    docker build -t kube-demo-frontend frontend

    cat frontend/frontend.yaml | sed "s/KEYCLOAK_HOST/$KEYCLOAK_HOST/" | \
    sed "s/BACKEND_HOST/$BACKEND_HOST/" | \
    kubectl create -f -

    cat frontend/frontend-ingress.yaml | sed "s/FRONTEND_HOST/$FRONTEND_HOST/" | \
    kubectl create -f - 

    xdg-open https://$FRONTEND_HOST

The frontend application should now be opened in your browser. Login with stian/pass. You should be able to invoke public and invoke secured, but not invoke admin. To be able to invoke admin go back to the Keycloak admin console and add the `admin` role to the user `stian`.
