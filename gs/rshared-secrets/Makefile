# For interactive prompt

SHELL = /bin/bash

# Help

help:
	@$(MAKE) -pRrq -f $(lastword $(MAKEFILE_LIST)) : 2>/dev/null \
		| awk -v RS= -F: '/^# File/,/^# Finished Make data base/ {if ($$1 !~ "^[#.]") {print $$1}}' \
		| egrep -v -e '^[^[:alnum:]]' -e '^$@$$' | sort -r

# Default settings

GCP_PROJECT := rshared
GCP_REGION := europe-west1
GCP_BUCKET := rshared-secrets

KMS_KEYRING := sites-mysql-kms-kr
KMS_CRYPTOKEY := sites-mysql-users-kms-ck

gcloud-config-$(GCP_PROJECT):
	gcloud config set project $(GCP_PROJECT)
	gcloud config set compute/region $(GCP_REGION)


SECRETS=$(shell \
	find . -type f -name '*.shasum' \
	| sed "s:./\(.*\).shasum:\1:g" \
)

$(addprefix encrypt-and-upload-,$(SECRETS)): encrypt-and-upload-%:
	gcloud kms encrypt \
		--location $(GCP_REGION) \
		--project $(GCP_PROJECT) \
		--keyring $(KMS_KEYRING) \
		--key $(KMS_CRYPTOKEY) \
		--plaintext-file $*.plain \
		--ciphertext-file $*.cipher
	gsutil cp $*.cipher gs://$(GCP_BUCKET)/$*.cipher
	shasum -a 512256 -b $*.cipher > $*.shasum
	gsutil cp $*.shasum gs://$(GCP_BUCKET)/$*.shasum

$(addprefix download-and-decrypt-,$(SECRETS)): download-and-decrypt-%:
	gsutil cp gs://$(GCP_BUCKET)/$*.cipher $*.cipher
	shasum -a 512256 -c $*.shasum
	gcloud kms decrypt \
		--location $(GCP_REGION) \
		--project $(GCP_PROJECT) \
		--keyring $(KMS_KEYRING) \
		--key $(KMS_CRYPTOKEY) \
		--plaintext-file $*.plain \
		--ciphertext-file $*.cipher

remove-secrets:
	@find . -type f -name '*.plain' -delete
	@find . -type f -name '*.cipher' -delete