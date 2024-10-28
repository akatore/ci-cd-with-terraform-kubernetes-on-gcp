# ci-cd-with-terraform-kubernetes-on-gcp



success:

![ScreenShot](https://github.com/user-attachments/assets/0a71e528-0f12-4df7-a96f-7b2224f8adbc)

Artifact Registry:

![image](https://github.com/user-attachments/assets/69aa5cdc-434c-40c5-b03e-315be17bdc46)

tf state file:

![image](https://github.com/user-attachments/assets/501452fc-1b71-4522-a424-09d564f33646)

GKE Cluster:

![image](https://github.com/user-attachments/assets/aac8a222-7730-417b-b63d-a1ec89475b2e)

Nodes (GCE):

![image](https://github.com/user-attachments/assets/d85a5dab-2545-456d-9fb9-12252ed9fc7c)

K8s service exposed as type LoadBalancer is accessible via External-IP
![image](https://github.com/user-attachments/assets/42180580-d108-4c6d-9ac1-f99a6cf4776c)

Workload management:
![image](https://github.com/user-attachments/assets/1083af2b-393b-4a7e-8f0c-a3640976569c)



<details> <summary>apis required </summary>
  
```
gcloud services enable \
  cloudresourcemanager.googleapis.com \
  container.googleapis.com \
  artifactregistry.googleapis.com \
  containerregistry.googleapis.com \
  containerscanning.googleapis.com  
  compute.googleapis.com
```
  <!-- ![image](https://github.com/user-attachments/assets/88c09f34-ae46-4678-8801-4d325c6aabeb) /-->
  
</details>

<details>

<summary> Set Workload Identity Federation to securely allow a non-GCP identity (like GitHub Actions) to authenticate and impersonate a service account in GCP. Breakdown of each step and how they interconnect:  </summary>

## Create a Service Account with Necessary IAM Roles
* **Create a Service Account**: Start by creating a service account with the permissions it will need to perform the actions you described:

```bash
gcloud iam service-accounts create SERVICE_ACCOUNT_NAME \
  --display-name "Service account for GitHub Actions"
```

<details> <summary> example </summary>

  ```
gcloud iam service-accounts create my-service-account \
  --description="Service account for application access" \
  --display-name="My Service Account" \
  --project="$PROJECT_ID"                                                                                                                                                                                                 
Created service account [my-service-account].
```
```
gcloud iam service-accounts list --project="$PROJECT_ID"                                                                                                             
DISPLAY NAME: My Service Account
EMAIL: my-service-account@$PROJECT_ID.iam.gserviceaccount.com
DISABLED: False
```
```
export SERVICE_ACCOUNT_EMAIL=my-service-account@$PROJECT_ID.iam.gserviceaccount.com
```

</details>

* **Assign Roles to the Service Account**: Grant the necessary roles to the service account for accessing GCP resources, such as:

  * `roles/container.admin` (to create GKE clusters)
  * `roles/compute.admin` (for Compute Engine access)
  * `roles/artifactregistry.admin` (for creating repositories and pushing images)
  * `roles/iam.serviceAccountTokenCreator` (for token creation)
```bash

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SERVICE_ACCOUNT_NAME@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/ROLE_NAME"
```

<details> <summary>role required on SA </summary>
  
  can be assigned using console or shell command 
  [create directly using this script](https://github.com/akatore/GCP-projects/blob/main/ci-cd-with-terraform-kubernetes-on-gcp/notes/scipt-to-assign-roles.sh) add as many needed.
  
![image](https://github.com/user-attachments/assets/c680dedd-2457-4997-930a-fe9dbfbce897)

![image](https://github.com/user-attachments/assets/4f531775-58eb-4a4d-b121-ba0d9f9dcade)


</details>

## Set Up Workload Identity Federation
**a. Create an Identity Pool**
* This step sets up an identity pool in which you define external identities and bind them to the GCP service account.
* You’ll specify attribute mappings for federated identity claims from GitHub Actions.
```bash
gcloud iam workload-identity-pools create IDENTITY_POOL_NAME \
  --project="PROJECT_ID" \
  --location="global" \
  --display-name="GitHub Actions Identity Pool"
```
**b. Create an Identity Provider for GitHub (OIDC)**
* Specify **GitHub** as the identity provider and configure OpenID Connect (OIDC) for token exchange.
```bash
gcloud iam workload-identity-pools providers create-oidc PROVIDER_NAME \
  --project="PROJECT_ID" \
  --location="global" \
  --workload-identity-pool="IDENTITY_POOL_NAME" \
  --display-name="GitHub Provider" \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="attribute.sub=assertion.sub,attribute.aud=assertion.aud,attribute.repository=assertion.repository" \
  --attribute-condition="attribute.repository==\"username/reponame\""
```
* **Issuer URI**: This URL allows GitHub to issue tokens for authentication via OIDC.
## Bind the Principal to the Service Account
**a. Grant the Service Account Impersonation Role**
* Next, bind the principal (from the identity pool) to have access to impersonate the service account:
```
bash
gcloud iam service-accounts add-iam-policy-binding SERVICE_ACCOUNT_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --project="PROJECT_ID" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/IDENTITY_POOL_NAME/attribute.repository/username/reponame"
```
<details> <summary> Create WIF & Identity Pool taking references from this GCP  Offical walkthorugh </summary>

## Watch the Tutorial

[![Watch the video](https://img.youtube.com/vi/ZgVhU5qvK1M/0.jpg)](https://www.youtube.com/watch?v=ZgVhU5qvK1M&t)

Click the image above to watch the tutorial on YouTube.

Outcome:

![image](https://github.com/user-attachments/assets/66fac5d1-60fe-4a67-9c3c-bcccf6a553a7)

![image](https://github.com/user-attachments/assets/46354fc0-2698-4cb1-ac21-1116bf6e277f)

</details>

## Summary
Now, when a GitHub Action runs, it will use OIDC to authenticate, matching conditions you set up in the identity pool and provider. With this, GitHub can impersonate the GCP service account and access resources securely without using JSON keys.

</details>




<details> <summary> Adding IAM Policy Binding to a Service Account for Workload Identity Allowing External Identity to Impersonate the Service Account </summary>
  
Grant the WIF identity `(principalSet://...)` permissions as Workload Identity User on the service account? Example from the README:

```
gcloud iam service-accounts add-iam-policy-binding "my-service-account@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}"
```

![image](https://github.com/user-attachments/assets/3a388ee1-47d3-432d-be73-3be7060a919d)

</details>

<details> <summary> Create a bucket in GCS for storing terraform state file </summary>

  ![image](https://github.com/user-attachments/assets/501452fc-1b71-4522-a424-09d564f33646)

</details>

<details> <summary> Get your GCP Project number for reference </summary>

```
gcloud projects describe ${PROJECT_ID}
```
</details>

<details> <summary> Add secrets to Github Repo </summary>

* GCP_PROJECT_ID
* GCP_TF_STATE_BUCKET
</details>

<details> <summary> Write GitHub Actions workflow for deploying our app to GKE using terraform </summary>

> Note: ADD secrets for repo for, `workload_identity_provider` and `service_account`
```
        workload_identity_provider: 'projects/$PROJECT_NUMBER/locations/global/workloadIdentityPools/{POOL_ID}/providers/{PROVIDER_ID}'
        service_account: 'tf-gke-test@$GCP_PROJECT_ID.iam.gserviceaccount.com'

example:
        workload_identity_provider: 'projects/88625s1781/locations/global/workloadIdentityPools/k8s-pool/providers/k8s-provider'
        service_account: 'tf-gke-test@$GCP_PROJECT_ID.iam.gserviceaccount.com'

```
as per below example WIF Identity Pool, ![image](https://github.com/user-attachments/assets/1c0b12c3-cba8-4034-9bd9-45f161600c13)

</details>
