# google-kubernetes-engine-app-deploy

Demonstration of deploying an application in Google Cloud using Google Kuberentes Engine (GKE).

![Final Environment](https://user-images.githubusercontent.com/3911650/48246112-53d23d00-e3ab-11e8-9e80-46c6366e0acc.png)

## Getting Started

1. Ensure the Following APIs are enabled (enable with gcloud services enable [service]):

    - container.googleapis.com

1. [Ensure the default Google APIs service account (used by deployment manager) has permission to create roles](https://cloud.google.com/deployment-manager/docs/configuration/set-access-control-resources):

    - In macOS/Linux:

        ```sh
        project_id=$(gcloud config list --format 'value(core.project)')
        project_number=$(gcloud projects list --filter "id:ca-labs" --format 'value(projectNumber)')
        gcloud projects add-iam-policy-binding $project_id \
          --member serviceAccount:$project_number@cloudservices.gserviceaccount.com \
          --role roles/iam.roleAdmin
        ```

    - In Windows (PowerShell):

        ```ps1
        $project_id = gcloud config list --format 'value(core.project)'
        $project_number = gcloud projects list --filter "id:ca-labs" --format 'value(projectNumber)'
        gcloud projects add-iam-policy-binding $project_id `
          --member serviceAccount:$project_number@cloudservices.gserviceaccount.com `
          --role roles/iam.roleAdmin
        ```

1. Deploy the deployment manager config in the `infrastructure` directory:

    ```sh
    gcloud deployment-manager deployments create lab --config infrastructure/deployment.yaml
    ```

1. Bind the Lab role to the student user or group:

    - In macOS/Linux:

        ```sh
        member="[GROUP_OR_USER]"
        project_id=$(gcloud config list --format 'value(core.project)')
        role=$(gcloud iam roles list --project $project_id \
                                     --filter "name:projects/$project_id/roles/studentrole*" \
                                     --format "value(name)")
        gcloud projects add-iam-policy-binding $project_id \
        --member $member  \
        --role $role
        ```

    - In Windows (PowerShell):

        ```ps1
        $member = "[GROUP_OR_USER]"
        $project_id = gcloud config list --format 'value(core.project)'
        $role = gcloud iam roles list --project $project_id `
                                      --filter "name:projects/$project_id/roles/studentrole*" `
                                      --format "value(name)"
        gcloud projects add-iam-policy-binding $project_id `
        --member $member  `
        --role $role
        ```

    An example of `[GROUP_OR_USER]` is `user:student@gmail.com`.

## Following Along

1. Start a Google Cloud Shell session.

1. Open the GKE section in the Cloud Console. Explore the different sections while waiting for the cluster to provision.

1. Click __Connect__ and run the provided command in Cloud Shell to install the kubeconfig configuration for `kubectl` to connect to the cluster.

1. Deploy a game application with the following command:

    ```sh
    kubectl run app --image=lrakai/tetris:1.0.0 --replicas=2 --labels="app=game"
    # Allow access from the internet
    kubectl expose deployment app --port=80 --target-port=80 --type=LoadBalancer
    ```

1. Output the external IP of the app service and navigate to the IP address to connect to the game:

    ```sh
    kubectl get service app
    ```

1. In the GKE Console, click __Marketplate__ in the left navigation pane.

1. Choose WordPress, click __Configure__ followed by __Deploy__ to deploy WordPress.

1. Allow external access by patching the WordPress service to be of `type: LoadBalancer`:

    ```sh
    kubectl patch svc "wordpress-1-wordpress-svc" --patch '{"spec": {"type": "LoadBalancer"}}'
    ```

1. Navigate to the external IP of the service to access WordPress:

    ```sh
    kubectl get service wordpress-1-wordpress-svc
    ```

## Tearing Down

When finished, remove the GCP resources with:

- In macOS/Linux:

    ```sh
    gcloud projects remove-iam-policy-binding $project_id \
        --member $member  \
        --role $role
    gcloud deployment-manager deployments delete -q lab
    ```

- In Windows (PowerShell):

    ```ps1
    gcloud projects remove-iam-policy-binding $project_id `
        --member $member  `
        --role $role
    gcloud deployment-manager deployments delete -q lab
    ```
