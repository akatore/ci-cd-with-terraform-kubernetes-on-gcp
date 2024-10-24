abhijeetkatore101@cloudshell:~ (gke-project-439604)$ gcloud auth activate-service-account gke-wsa@gke-project-439604.iam.gserviceaccount.com --key-file=gke-project-439604-55210b49e482.json                              
Activated service account credentials for: [gke-wsa@gke-project-439604.iam.gserviceaccount.com]
abhijeetkatore101@cloudshell:~ (gke-project-439604)$ 


Enable necessary API for project

IAM Service Account Credentials API

Grant the roles/iam.serviceAccountTokenCreator role to the GitHub Actions identity for the service account gke-wsa@<YOUR_PROJECT_ID>.iam.gserviceaccount.com.

gcloud iam service-accounts add-iam-policy-binding gke-wsa@$PROJECT_ID.iam.gserviceaccount.com \
    --member="serviceAccount:gke-wsa@gke-project-439604.iam.gserviceaccount.com" \
    --role="roles/iam.workloadIdentityUser"


gcloud iam service-accounts add-iam-policy-binding gke-wsa@$PROJECT_ID.iam.gserviceaccount.com \
    --member="serviceAccount:gke-wsa@gke-project-439604.iam.gserviceaccount.com" \
    --role="roles/iam.workloadIdentityUser"


abhijeetkatore101@cloudshell:~ (gke-project-439604)$ gcloud iam service-accounts add-iam-policy-binding gke-wsa@$PROJECT_ID.iam.gserviceaccount.com \
    --member="serviceAccount:gke-wsa@gke-project-439604.iam.gserviceaccount.com" \
    --role="roles/iam.workloadIdentityUser"
Updated IAM policy for serviceAccount [gke-wsa@gke-project-439604.iam.gserviceaccount.com].
bindings:
- members:
  - principalSet://iam.googleapis.com/projects/189792038322/locations/global/workloadIdentityPools/k8s-pool/attribute.repository/akatore/GCP-projects/
  - serviceAccount:gke-wsa@gke-project-439604.iam.gserviceaccount.com
  role: roles/iam.workloadIdentityUser
etag: BwYlObhZKUE=
version: 1



gcloud iam service-accounts add-iam-policy-binding "my-service-account@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}"

gcloud iam service-accounts add-iam-policy-binding "my-service-account@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/189792038322/attribute.repository/akatore/GCP-projects"



gcloud iam workload-identity-pools providers create-oidc k8s-providero \
  --workload-identity-pool="k8s-pool" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --location="global"


enable artifact registry API:
- Artifact Registry API
+ Container Registry API


https://www.youtube.com/watch?v=ZgVhU5qvK1M&t=517s

https://github.com/google-github-actions/auth/issues/426#issuecomment-2227887783
