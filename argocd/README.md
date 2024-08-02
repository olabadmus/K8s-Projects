# BootStrap K8s with ArgoCD


## Table of Contents

1. [Introduction](#introduction)
2. [Clone the Repository](#clone-the-repository)
3. [Provision EKS Cluster](#provision-eks-cluster)
    - [Initialize Terraform](#initialize-terraform)
    - [Plan Terraform](#Plan-terraform)
    - [Apply Terraform Configuration](#apply-terraform-configuration)
4. [Update EKS Configuration](#update-eks-configuration)
5. [Apply Manifest Files](#apply-manifest-files)
6. [Bootstrapping with ArgoCD](#bootstrapping-with-argocd)
    - [Add a Namespace for ArgoCD](#add-a-namespace-for-argocd)
    - [Install ArgoCD](#install-argocd)
    - [Confirm ArgoCD Pods](#confirm-argocd-pods)
    - [Map Port for ArgoCD Access](#map-port-for-argocd-access)
    - [Retrieve ArgoCD Admin Password](#retrieve-argocd-admin-password)
    - [Log in to ArgoCD](#log-in-to-argocd)
    - [Connect GitHub Repository to ArgoCD](#connect-github-repository-to-argocd)
    - [Add Your Cluster to ArgoCD](#add-your-cluster-to-argocd)
    - [Create and Sync Your Application](#create-and-sync-your-application)
7. [Conclusion](#conclusion)

### Introduction
In this article, we will learn how to bootstrap our Kubernetes cluster with ArgoCD. To achieve this, we need a running cluster which can either be EKS, GKS, or AKS, our manifest files which are inside our GitHub repository, and an ArgoCD account.


### Clone the repository

```sh
git clone https://github.com/philcz16/manifests
```
![image](https://github.com/user-attachments/assets/d8cdc2e5-f48d-4d2e-950b-b7036511232c)


cd into the eks folder which contains the Terraform files to provision your cluster on AWS:

```sh
terraform init
terraform plan
terraform apply
```
![image](https://github.com/user-attachments/assets/5c42ad05-baa3-4cc5-a44f-f0d710f2b28a)
![image](https://github.com/user-attachments/assets/5d13f81e-374a-49c1-a3ba-415aa4aa68c8)
![image](https://github.com/user-attachments/assets/528b9002-5ccb-4a13-bde1-7779347bce54)

![image](https://github.com/user-attachments/assets/4cd814ed-cbc7-4c4b-b190-aa6b73b1c00f)
![image](https://github.com/user-attachments/assets/9f1f4622-0e7d-4895-b93d-38e822e1c2a7)



### Update EKS Configuration

After our EKS configuration has been successfully applied to AWS, update it using the following command so our manifest files can be applied to the cluster:

```sh
aws eks update-kubeconfig --name demo-cluster --region us-east-1
```
![image](https://github.com/user-attachments/assets/b397b6e5-3f1b-4103-aaef-705418785380)


### Apply Manifest Files

After updating your cluster to receive manifests configuration, apply the files in the manifest folder with:

```sh
kubectl apply -f app.yaml
```
![image](https://github.com/user-attachments/assets/3f03b75f-1772-4567-b3d0-f33e1a4dbb5f)

### Bootstrapping with ArgoCD

Once your manifest files have been applied, the next step is to bootstrap them to ArgoCD to enhance continuous delivery. Follow these steps:

1. **Add a namespace for ArgoCD:**

    ```sh
    kubectl create namespace argocd
    ```

2. **Install ArgoCD:**

    ```sh
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```
    ![image](https://github.com/user-attachments/assets/926dccdd-c482-43fb-bce1-4b8171552acb)


3. **Confirm ArgoCD Pods:**

    After installation, wait for a while for the pods to be ready and confirm it using:

    ```sh
    kubectl get pods -n argocd
    ```
    ![image](https://github.com/user-attachments/assets/04644a70-50ca-4299-a07e-9f7f1d43661a)


4. **Map Port for ArgoCD Access:**

    When the pods are ready, map your port to access ArgoCD in the browser:

    ```sh
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```
    ![image](https://github.com/user-attachments/assets/8b861a94-d9f9-43a1-877f-38d391917489)


5. **Retrieve ArgoCD Admin Password:**

    Upon access, you will be required to log in with a username and a password. The username is `admin`, and the password can be retrieved using the following command:

    ```sh
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```
    ![image](https://github.com/user-attachments/assets/6b4c38d6-9ede-4e5f-b891-41a1a1cae7d2)


6. **Log in to ArgoCD:**

    When the password has been retrieved, log in to ArgoCD with:

    ```sh
    argocd login localhost:8080
    ```
    ![image](https://github.com/user-attachments/assets/9e6e17f5-f3dc-45d0-a646-e6abaf862894)
   ![image](https://github.com/user-attachments/assets/d32dc3e5-35ce-4d05-891b-71a9457a669e)



8. **Connect GitHub Repository to ArgoCD:**

    Once logged in successfully, connect the GitHub repo that contains the manifest with the following command:

    ```sh
    argocd repo add https://github.com/username/repourl --username <your-github-username> --password <your-personal-access-token>
    ```
    ![image](https://github.com/user-attachments/assets/798f9182-48d9-4780-a624-0e9263e4698c)


    Note: To get your GitHub password, use your GitHub token, which can be generated in developer’s settings.

9. **Add Your Cluster to ArgoCD:**

    Once your repo has been connected successfully, add your cluster to the ArgoCD server using the following command:

    ```sh
    kubectl config get-contexts
    argocd cluster add <context-name>
    ```
    ![image](https://github.com/user-attachments/assets/3fa96873-7a60-4eb6-829d-de5fb7d5c280)


10. **Create and Sync Your Application:**

    Once the cluster has been added successfully, proceed to create your app and configure your ArgoCD using:

    ```sh
    argocd app create tundeapp \
       --repo https://github.com/olabadmus/K8s-Projects.git \
       --path manifests \
       --dest-server https://kubernetes.default.svc \
       --dest-namespace argocd
    ```

    ![image](https://github.com/user-attachments/assets/fc25882e-0c2a-47c3-b567-346fe5327747)


11. **Sync Your Application:**

    Finally, sync your app using the following command:

    ```sh
    argocd app sync newapp
    ```

    ![image](https://github.com/user-attachments/assets/2666d3e5-3e05-4da9-a7c5-cf59809310fd)
    ![image](https://github.com/user-attachments/assets/637dd281-c02a-4280-b652-20a8f8bcb69d)



Once the sync is successful, you can log in to your ArgoCD via the browser to check it out.

![image](https://github.com/user-attachments/assets/fec978a1-f2e0-4b6d-8862-1ac8afd47a7f)


## Conclusion

By following these steps, you have set up a Kubernetes cluster, applied your manifest files, and integrated ArgoCD for continuous delivery. This setup not only streamlines your deployment process but also enhances the manageability and scalability of your applications.
