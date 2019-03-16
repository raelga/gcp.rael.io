GCP_PROJECT := rshared
GCP_INSTANCE := chell
GCP_ZONE := europe-west1-b

GCP_CFG = --project $(GCP_PROJECT) --zone $(GCP_ZONE)

ssh:
	gcloud compute ssh $(GCP_INSTANCE) $(GCP_CFG)

start:
	gcloud compute instances start $(GCP_INSTANCE) $(GCP_CFG)

stop:
	gcloud compute instances stop $(GCP_INSTANCE) $(GCP_CFG)

pump: stop .update-machine-type-to-n1-standard-2 start

dump: stop .update-machine-type-to-f1-micro start

.update-machine-type-to-%:
	gcloud compute instances set-machine-type $(GCP_INSTANCE) $(GCP_CFG) --machine-type $*
