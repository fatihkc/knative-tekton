# GitOps with Tekton and ArgoCD

## Technologies

* OpenShift
* Ansible
* Knative
* Tekton
* ArgoCD
* Quarkus

## Requirements

* OpenShift
* Ansible

## Initial configuration

Install Kubernetes modules.

    ansible-galaxy collection install community.kubernetes
    
Login to your OpenShift cluster

    oc login --token={your-users-token} --server={server-domain}:{server-port}
    
Change GitHub credentials at **roles/tekton/templates/github.credentials**

    apiVersion: v1
    kind: Secret
    metadata:
      name: github-credentials
      annotations:
        tekton.dev/git-0: https://github.com
    type: kubernetes.io/basic-auth
    stringData:
      username: fatihkc
      password: {your-github-password}

## Start Ansible playbooks

Firstly, start your tekton playbook for initial configuration.
    
    ansible-playbook tekton.yaml -v
    
Start pipeline with your variables

    cat tekton/pipelines/knative-pipeline-run.yaml | \
      SOURCE_REPO=https://github.com/fatihkc/quarkus-hello-world.git \
      COMMIT=9ce90240f96a9906b59225fec16d830ab4f3fe12 \
      SHORT_COMMIT=9ce9024 \
      DEPLOYMENT_REPO=https://github.com/fatihkc/pipeline-test.git \
      IMAGES_NS=cicd envsubst | \
      oc create -f - -n cicd

Check your pipeline tasks. If everything looks fine then start ArgoCD pipeline.

    ansible-playbook argo.yaml -v
    
Get your route for accessing ArgoCD. Use your OpenShift credentials for access. This can take a while.
    
    oc get routes argocd-server -n cicd
    
Now you need to see your deployments. Start developing your application!