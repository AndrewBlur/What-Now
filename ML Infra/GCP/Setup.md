# GCP Deployment Runbook — ML Model Serving API

**Project:** `aiinfra-andrew-007`  
**Region:** `asia-south1` (Mumbai)  
**VM zone:** `asia-south1-a`  
**VM name:** `ml-inference-vm`  
**Container image:** `image-classifier:latest`  
**Registry:** Google Artifact Registry  
**Application:** FastAPI + PyTorch ResNet18

---

## 1. Deployment architecture

The application was built and containerized locally on Windows, uploaded to Google Artifact Registry, and deployed to a Compute Engine virtual machine.

```
Windows Development Machine
          |
          | Docker build
          v
   Docker Image
          |
          | Docker tag + push
          v
 Google Artifact Registry
          |
          | Docker pull
          v
  Compute Engine VM
  Ubuntu 24.04 LTS
          |
          v
   Docker Container
          |
          v
  FastAPI :8000
          |
          v
   Image Prediction
```

The VM initially exposed the API only to its own loopback interface. We subsequently changed the Docker port binding and created a GCP firewall rule to make the API publicly reachable.

---

## 2. Configure the Google Cloud project

### Set the active project

```
gcloud config set project aiinfra-andrew-007
```

**What it does:** Sets the default GCP project for subsequent `gcloud` commands.

**Why we used it:** We needed to ensure that the VM, Artifact Registry repository, IAM permissions, and firewall rules were created in the intended project.

### Verify the active project

```
gcloud config get-value project
```

Expected output:

```
aiinfra-andrew-007
```

### Verify the authenticated account

```
gcloud auth list
```

**What it does:** Displays authenticated Google accounts and identifies the active account.

**Issue encountered:** We needed to establish which Google account was being used before creating cloud resources.

### List available projects

```
gcloud projects list
```

**What it does:** Lists GCP projects accessible to the authenticated account.

**Why we used it:** We checked the available projects before proceeding with deployment.

### Check existing Compute Engine instances

```
gcloud compute instances list
```

**What it does:** Lists Compute Engine instances in the active project.

**Issue encountered:** The project initially had no VM instances, so we needed to provision one.

---

## 3. Create the Compute Engine VM

```
gcloud compute instances create ml-inference-vm `
  --project=aiinfra-andrew-007 `
  --zone=asia-south1-a `
  --machine-type=e2-standard-2 `
  --image-family=ubuntu-2404-lts-amd64 `
  --image-project=ubuntu-os-cloud `
  --boot-disk-size=30GB `
  --boot-disk-type=pd-balanced
```

### What each argument means

|Argument|Purpose|
|---|---|
|`ml-inference-vm`|Name of the virtual machine|
|`--project`|GCP project that owns the VM|
|`--zone`|Physical cloud zone for the VM|
|`--machine-type=e2-standard-2`|VM with 2 vCPUs and approximately 8 GiB RAM|
|`--image-family`|Ubuntu 24.04 LTS operating system image family|
|`--image-project`|Project containing the official Ubuntu image|
|`--boot-disk-size=30GB`|Sets the boot disk size|
|`--boot-disk-type=pd-balanced`|Uses a balanced persistent disk|

**Why we used it:** The model-serving application needed a cloud machine capable of running the Docker container and performing CPU-based image inference.

### Verify the VM

```
gcloud compute instances list
```

### Connect to the VM

```
gcloud compute ssh ml-inference-vm --zone=asia-south1-a
```

**What it does:** Establishes an SSH connection to the Linux VM using Google Cloud's SSH integration.

**Why we used it:** We needed to install Docker, download the container image, and run the application remotely.

---

## 4. Inspect the VM's resources

These commands run inside the Linux SSH session.

### Check the operating system

```
cat /etc/os-release
```

**Purpose:** Confirms the Linux distribution and version. Our VM runs Ubuntu 24.04 LTS.

### Check available CPU cores

```
nproc
```

**Purpose:** Reports the number of processing units visible to the operating system. Our VM reported `2`.

### Check memory

```
free -h
```

**Purpose:** Displays RAM and swap usage in human-readable units.

### Check disk usage

```
df -h
```

**Purpose:** Shows available and used disk space for mounted filesystems.

To inspect only the root filesystem:

```
df -h /
```

**Issue encountered:** We accidentally ran `df -h/`, which produced an invalid-option error because the space between `-h` and `/` was missing.

**Lesson:** Linux command-line arguments and paths must be separated correctly.

---

## 5. Install and verify Docker on the VM

### Update package metadata

```
sudo apt update
```

**What it does:** Refreshes Ubuntu's package indexes.

**Why we used it:** We needed current package metadata before installing Docker.

### Install Docker

```
sudo apt install -y docker.io
```

**What it does:** Installs Ubuntu's packaged Docker Engine.

**Issue encountered:** Docker was not initially installed on the VM.

**Resolution:** Installed it using Ubuntu's package manager.

### Start Docker and enable automatic startup

```
sudo systemctl enable --now docker
```

**What it does:**

- `enable` configures Docker to start at boot.
    
- `--now` starts the service immediately.
    

### Verify the Docker version

```
sudo docker --version
```

### Inspect Docker's configuration

```
sudo docker info
```

**Purpose:** Checks whether Docker is operational and displays its runtime configuration.

### Run a test container

```
sudo docker run --rm hello-world
```

**What it does:** Downloads and runs Docker's test image, then removes the stopped container.

**Success condition:** Docker prints a message confirming that the installation is working.

### Optional: Allow the user to run Docker without `sudo`

```
sudo usermod -aG docker "$USER"
```

Then exit and reconnect:

```
exit
```

From Windows PowerShell:

```
gcloud compute ssh ml-inference-vm --zone=asia-south1-a
```

Test:

```
docker run --rm hello-world
```

**Security note:** Membership in the `docker` group effectively grants root-level control of the machine. Keep this in mind when deciding which accounts to add.

---

## 6. Build the Docker image locally

Run these commands in Windows PowerShell.

### Navigate to the project

```
cd "$HOME\ai-infra-model-serving"
```

### Build the image

```
docker build -t image-classifier:latest .
```

**What it does:**

- Reads the project's `Dockerfile`.
    
- Builds the application image and its dependencies.
    
- Tags the resulting image as `image-classifier:latest`.
    

The final `.` means the current directory is the Docker build context.

### List local images

```
docker images
```

**Purpose:** Confirms that the expected image exists locally before uploading it.

---

## 7. Enable Artifact Registry and create a repository

Artifact Registry stores versioned Docker images so that cloud machines can retrieve them.

### Enable the Artifact Registry API

```
gcloud services enable artifactregistry.googleapis.com
```

**What it does:** Enables the Artifact Registry API for the selected project.

**Why we used it:** We needed a managed container registry for transferring the image from the local development machine to the VM.

### Create the Docker repository

```
gcloud artifacts repositories create ml-inference `
  --project=aiinfra-andrew-007 `
  --repository-format=docker `
  --location=asia-south1 `
  --description="Docker images for ML model serving"
```

**What it does:** Creates a Docker repository named `ml-inference` in the Mumbai region.

**Important distinction:** The repository is separate from the Docker image. The repository is the remote storage location; the image is the artifact stored inside it.

If the repository already exists, inspect it instead of creating another one:

```
gcloud artifacts repositories list `
  --project=aiinfra-andrew-007 `
  --location=asia-south1
```

---

## 8. Authenticate Docker and push the image

### Configure Docker authentication on Windows

```
gcloud auth configure-docker asia-south1-docker.pkg.dev
```

**What it does:** Configures Docker to use Google Cloud credentials when accessing Artifact Registry in the specified regional hostname.

### Tag the image with its remote registry path

```
docker tag image-classifier:latest asia-south1-docker.pkg.dev/aiinfra-andrew-007/ml-inference/image-classifier:latest
```

**What it does:** Assigns a fully qualified registry name to the local image. It does not upload the image yet.

The naming structure is:

```
REGION-docker.pkg.dev/PROJECT/REPOSITORY/IMAGE:TAG
```

For our image:

```
asia-south1-docker.pkg.dev/aiinfra-andrew-007/ml-inference/image-classifier:latest
```

### Push the image

```
docker push asia-south1-docker.pkg.dev/aiinfra-andrew-007/ml-inference/image-classifier:latest
```

**What it does:** Uploads the image layers to Artifact Registry.

**Success condition:** Docker completes the upload and prints the resulting image digest.

---

## 9. Fix the Artifact Registry authentication issue on the VM

### Error encountered

The VM initially returned:

```
Unauthenticated request. Unauthenticated requests do not have permission
"artifactregistry.repositories.downloadArtifacts"
```

### Root cause

Two separate requirements must be satisfied:

1. The VM's service account needs permission to download artifacts.
    
2. Docker must authenticate to the registry.
    

Granting IAM permission does not, by itself, guarantee that Docker sends authenticated registry requests.

### Identify the VM's service account

Run inside the VM:

```
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/email
```

Our VM returned:

```
563681809389-compute@developer.gserviceaccount.com
```

This is the VM's attached default Compute Engine service account.

### Grant Artifact Registry Reader permission

Run in Windows PowerShell:

```
gcloud artifacts repositories add-iam-policy-binding ml-inference `
  --project=aiinfra-andrew-007 `
  --location=asia-south1 `
  --member="serviceAccount:563681809389-compute@developer.gserviceaccount.com" `
  --role="roles/artifactregistry.reader"
```

**What it does:** Grants the VM's service account permission to read artifacts from this repository.

### Verify the repository IAM policy

```
gcloud artifacts repositories get-iam-policy ml-inference `
  --project=aiinfra-andrew-007 `
  --location=asia-south1
```

We verified that the service account was listed under:

```
roles/artifactregistry.reader
```

### Inspect the VM's access scopes

```
gcloud compute instances describe ml-inference-vm `
  --zone=asia-south1-a `
  --format="yaml(serviceAccounts)"
```

**What it does:** Displays the service account attached to the VM and its OAuth access scopes.

The original VM had the `devstorage.read_only` scope. We changed the VM to use the broader `cloud-platform` OAuth scope while relying on IAM roles to control actual resource access.

### Stop the VM

```
gcloud compute instances stop ml-inference-vm --zone=asia-south1-a
```

**Why:** The VM must be stopped before changing its attached service-account configuration in this workflow.

### Set the access scope

```
gcloud compute instances set-service-account ml-inference-vm `
  --zone=asia-south1-a `
  --scopes=cloud-platform
```

### Start the VM

```
gcloud compute instances start ml-inference-vm --zone=asia-south1-a
```

Then reconnect:

```
gcloud compute ssh ml-inference-vm --zone=asia-south1-a
```

**Important:** The `cloud-platform` scope does not grant every permission by itself. IAM permissions remain the primary authorization control.

### Why the pull still failed

Even after correcting the IAM policy and access scope, Docker continued making unauthenticated requests. The remaining issue was registry authentication from the Docker client on the VM.

### Obtain a temporary service-account access token

Inside the VM:

```
TOKEN=$(curl -s \
  -H "Metadata-Flavor: Google" \
  "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" \
  | python3 -c "import json,sys; print(json.load(sys.stdin)['access_token'])")
```

**What it does:** Requests a short-lived OAuth access token from the VM metadata server.

The token is stored in the shell variable `TOKEN`.

Verify that a token was obtained without printing it:

```
if [ -n "$TOKEN" ]; then
  echo "Token obtained"
else
  echo "Token retrieval failed"
fi
```

### Authenticate Docker

```
echo "$TOKEN" | sudo docker login \
  -u oauth2accesstoken \
  --password-stdin \
  https://asia-south1-docker.pkg.dev
```

**What it does:**

- Uses the OAuth access token as the Docker registry password.
    
- Supplies it through standard input instead of placing it directly in the command arguments.
    
- Authenticates Docker to the regional Artifact Registry endpoint.
    

Expected output:

```
Login Succeeded
```

### Pull the image

```
sudo docker pull asia-south1-docker.pkg.dev/aiinfra-andrew-007/ml-inference/image-classifier:latest
```

**Result:** The image downloaded successfully after Docker was authenticated.

**Security note:** Never print, commit, or share the token. OAuth tokens expire, so this manual authentication is a troubleshooting solution rather than the ideal long-term credential-management approach. For a persistent deployment, prefer the supported Google Cloud credential helper or another approved credential mechanism.

---

## 10. Run the application container on the VM

### Start the container privately

Our initial container command was:

```
sudo docker run -d \
  --name image-classifier \
  --restart unless-stopped \
  -p 127.0.0.1:8000:8000 \
  asia-south1-docker.pkg.dev/aiinfra-andrew-007/ml-inference/image-classifier:latest
```

### What the options mean

|Option|Purpose|
|---|---|
|`-d`|Runs the container in the background|
|`--name image-classifier`|Gives the container a recognizable name|
|`--restart unless-stopped`|Restarts it after many Docker or host restarts unless explicitly stopped|
|`-p 127.0.0.1:8000:8000`|Publishes container port 8000 only on the VM's loopback interface|
|Image path|Identifies the image to run|

The container's port is `8000`; the host port is also `8000`.

### Inspect running containers

```
sudo docker ps
```

### Inspect application logs

```
sudo docker logs --tail 100 image-classifier
```

**Purpose:** Helps diagnose startup failures, model download problems, and application errors.

### Test the health endpoint locally

```
curl http://127.0.0.1:8000/health
```

**Result:** The health endpoint responded successfully.

---

## 11. Make the API publicly reachable

**Warning:** The following configuration makes the API reachable from the public internet. Add authentication, rate limiting, and appropriate request limits before using this for a real service.

### Remove the private container

```
sudo docker rm -f image-classifier
```

**What it does:** Stops and removes the existing container so it can be recreated with a different port binding. It does not remove the Docker image.

### Recreate the container with a public port binding

```
sudo docker run -d \
  --name image-classifier \
  --restart unless-stopped \
  -p 8000:8000 \
  asia-south1-docker.pkg.dev/aiinfra-andrew-007/ml-inference/image-classifier:latest
```

**Important difference:**

- `127.0.0.1:8000:8000` — accessible only through the VM's loopback interface.
    
- `8000:8000` — publishes port 8000 on all host network interfaces.
    

The second form allows external traffic to reach Docker, provided the cloud and host firewalls permit it.

### Create the GCP firewall rule

Run in Windows PowerShell:

```
gcloud compute firewall-rules create allow-image-classifier-8000 `
  --project=aiinfra-andrew-007 `
  --network=default `
  --direction=INGRESS `
  --action=ALLOW `
  --rules=tcp:8000 `
  --source-ranges=0.0.0.0/0 `
  --target-tags=ml-inference-public
```

**What it does:**

- Creates an ingress firewall rule.
    
- Allows TCP traffic on port 8000.
    
- Allows traffic from all IPv4 addresses.
    
- Applies the rule only to VM instances with the `ml-inference-public` network tag on the specified VPC network.
    

### Add the matching network tag

```
gcloud compute instances add-tags ml-inference-vm `
  --project=aiinfra-andrew-007 `
  --zone=asia-south1-a `
  --tags=ml-inference-public
```

**What it does:** Adds the network tag required for the firewall rule to target this VM.

**Why both commands are required:** Creating the rule defines the traffic policy; adding the tag makes the rule apply to the intended instance.

### Test the API from Windows

Health check:

```
Invoke-RestMethod http://8.231.126.193:8000/health
```

Swagger UI:

```
http://8.231.126.193:8000/docs
```

**Result:** The public health endpoint and API documentation became accessible using the VM's external IP.

**Note:** An external IP address can change if the VM is stopped and restarted unless a static external IP is reserved. If the address changes, obtain the current address with:

```
gcloud compute instances describe ml-inference-vm `
  --zone=asia-south1-a `
  --format="get(networkInterfaces[0].accessConfigs[0].natIP)"
```

---

## 12. Common issues and resolutions

|Issue|Cause|Resolution|
|---|---|---|
|No VM instances listed|No Compute Engine VM existed in the project|Created `ml-inference-vm`|
|`docker: command not found`|Docker was not installed on Ubuntu|Installed `docker.io`|
|`df: invalid option -- '/'`|Missing space in `df -h/`|Used `df -h /`|
|Artifact Registry API unavailable|Required API needed to be enabled|Enabled `artifactregistry.googleapis.com`|
|`Unauthenticated request` during pull|Docker did not authenticate to Artifact Registry|Retrieved a VM access token and logged Docker in|
|Artifact download permission denied|Service account needed repository read permission|Granted `roles/artifactregistry.reader`|
|VM still could not access the registry|IAM, OAuth scope, and Docker authentication are separate concerns|Checked IAM and access scopes, then authenticated Docker explicitly|
|Public endpoint inaccessible|Container port binding, firewall policy, and network tag must align|Recreated the container with `-p 8000:8000`, created the firewall rule, and added the tag|
|`google-cloud-cli` package unavailable|The package was not present in the default Ubuntu package sources|Used the VM metadata server token instead; no CLI installation was required|

---

## 13. Useful operational commands

### View running containers

```
sudo docker ps
```

### View all containers, including stopped ones

```
sudo docker ps -a
```

### Follow application logs

```
sudo docker logs -f image-classifier
```

### Restart the container

```
sudo docker restart image-classifier
```

### Check container resource usage

```
sudo docker stats image-classifier
```

### Stop the VM when finished

Run in Windows PowerShell:

```
gcloud compute instances stop ml-inference-vm --zone=asia-south1-a
```

**Why:** A running VM incurs compute charges. Stopping it when you are not using it reduces compute costs, although persistent disks and some other resources may continue to incur charges.

Start it again when needed:

```
gcloud compute instances start ml-inference-vm --zone=asia-south1-a
```

Then reconnect:

```
gcloud compute ssh ml-inference-vm --zone=asia-south1-a
```

---

## 14. Current deployment status

|Component|Status|
|---|---|
|GCP project configured|Complete|
|Compute Engine VM created|Complete|
|Ubuntu VM inspected|Complete|
|Docker installed and tested|Complete|
|Artifact Registry repository created|Complete|
|Docker image built and pushed|Complete|
|VM service-account IAM permission configured|Complete|
|VM OAuth access scope updated|Complete|
|Docker authenticated to Artifact Registry|Complete|
|Image pulled onto VM|Complete|
|FastAPI container started|Complete|
|Local health check|Successful|
|Public port binding configured|Complete|
|GCP firewall rule and network tag configured|Complete|
|Public API access|Working|

## 15. Next recommended improvements

The current deployment is suitable for learning, but it is not yet a production-ready serving platform.

Recommended next steps:

1. **Security:** Put the API behind HTTPS and add authentication, rate limiting, and appropriate access controls.
    
2. **Health and recovery:** Configure container health checks and test restart behavior.
    
3. **Performance:** Measure latency and throughput under concurrent requests.
    
4. **Observability:** Track request errors, inference latency, CPU, memory, and container logs.
    
5. **Deployment hygiene:** Pin image versions or deploy by image digest instead of relying on the mutable `latest` tag.
    
6. **Cost management:** Stop the VM when not in use and review persistent-disk and networking charges.
    
7. **Infrastructure as code:** Recreate the VM, firewall rules, IAM bindings, and other resources from a documented configuration rather than relying exclusively on manual commands.
    

---

**End of runbook**

This document records the commands and troubleshooting steps used during this deployment. Before reusing it in another environment, verify the project ID, region, zone, service account, image path, and external IP.