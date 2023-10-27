# Welcome to the ```satusehat-integration``` library wiki!

## Quick Installation

### 1. Composer Require

```bash
composer require ivanwilliammd/satusehat-integration
```

### 2. Publish Config

```bash
php artisan vendor:publish --provider="Satusehat\Integration\SatusehatIntegrationServiceProvider" --tag=config
```

### 3. Publish Migration

```bash
php artisan vendor:publish --provider="Satusehat\Integration\SatusehatIntegrationServiceProvider" --tag=migrations
```

### 4. Run Migration

```bash
php artisan migrate
```

### 5. Publish ICD-10 Migration, Seeder, and CSV file
```bash
# Run to publish all assets regarding ICD-10
php artisan vendor:publish --provider="Satusehat\Integration\SatusehatIntegrationServiceProvider" --tag=icd10

# Run to migrate the ICD-10 table
php artisan migrate

# Run to seed the data
php artisan db:seed --class=Icd10Seeder
```


### 6. Setup the environment
Isilah Organization ID, Client ID dan Client Secret yang diberikan oleh SATUSEHAT di file .env

```env
SATUSEHAT_ENV=xxxxxx (DEV/STG/PROD)

SATUSEHAT_AUTH_DEV=https://api-satusehat-dev.dto.kemkes.go.id/oauth2/v1
SATUSEHAT_FHIR_DEV=https://api-satusehat-dev.dto.kemkes.go.id/fhir-r4/v1

SATUSEHAT_AUTH_STG=https://api-satusehat-stg.dto.kemkes.go.id/oauth2/v1
SATUSEHAT_FHIR_STG=https://api-satusehat-stg.dto.kemkes.go.id/fhir-r4/v1

SATUSEHAT_AUTH_PROD=https://api-satusehat.kemkes.go.id/oauth2/v1
SATUSEHAT_FHIR_PROD=https://api-satusehat.kemkes.go.id/fhir-r4/v1

ORGID_DEV=xxxxxx
CLIENTID_DEV=xxxxxx
CLIENTSECRET_DEV=xxxxxx

ORGID_STG=xxxxxx
CLIENTID_STG=xxxxxx
CLIENTSECRET_STG=xxxxxx

ORGID_PROD=xxxxxx
CLIENTID_PROD=xxxxxx
CLIENTSECRET_PROD=xxxxxx
```

Now, you are good to go!