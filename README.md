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



<details> <summary>roles required </summary>
```
gcloud services enable \
  cloudresourcemanager.googleapis.com \
  container.googleapis.com \
  artifactregistry.googleapis.com \
  containerregistry.googleapis.com \
  containerscanning.googleapis.com  
  compute.googleapis.com
```
  ![image](https://github.com/user-attachments/assets/88c09f34-ae46-4678-8801-4d325c6aabeb)
  
</details>
