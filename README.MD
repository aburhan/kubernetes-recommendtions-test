# Action
# Setting up Workload Identity Federation

https://github.com/marketplace/actions/authenticate-to-google-cloud#setup

# create service account 
export PROJECT_ID="my-project" # update with your value


# Enable the api
gcloud services enable bigquery.googleapis.com \
  --project "${PROJECT_ID}" 

# Create a Workload Identity Pool:
gcloud iam workload-identity-pools create "github-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="Demo pool"

# Get the full ID of the Workload Identity Pool:
export WORKLOAD_IDENTITY_POOL_ID=$(gcloud iam workload-identity-pools describe "github-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --format="value(name)")

# Create a Workload Identity Provider in that pool:
gcloud iam workload-identity-pools providers create-oidc "my-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github-pool" \
  --display-name="Demo provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
# uncomment for all repos --attribute-mapping="google.subject=assertion.sub,attribute.repository_owner=assertion.repository_owner"
  --issuer-uri="https://token.actions.githubusercontent.com"

# Allow authentications from the Workload Identity Provider originating from your repository to impersonate the Service Account created above:

# TODO(developer): Update this value to your GitHub repository.
export REPO="aburhan/kubernetes-recommendtions-test"

# Bind the service account
gcloud iam service-accounts add-iam-policy-binding "github-actions@github-action-393418.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}"
  # all repos --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository_owner/${OWNER}"

# Extract the Workload Identity Provider resource name
export workload_identity_provider=$(gcloud iam workload-identity-pools providers describe "my-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github-pool" \
  --format="value(name)")

# Use this value as the workload_identity_provider value in your GitHub Actions YAML.
  projects/168241442549/locations/global/workloadIdentityPools/github-pool/providers/my-provider