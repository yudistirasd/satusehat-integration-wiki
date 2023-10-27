# Welcome to the ```satusehat-integration``` library wiki!

## Quick Installation

### Composer Require

```bash
composer require ivanwilliammd/satusehat-integration
```

### Publish Config

```bash
php artisan vendor:publish --provider="Satusehat\Integration\SatusehatIntegrationServiceProvider" --tag=config
```

### Publish Migration

```bash
php artisan vendor:publish --provider="Satusehat\Integration\SatusehatIntegrationServiceProvider" --tag=migrations
```

### Run Migration

```bash
php artisan migrate
```

### Publish ICD-10 Migration, Seeder, and CSV file
```bash
# Run to publish all assets regarding ICD-10
php artisan vendor:publish --provider="Satusehat\Integration\SatusehatIntegrationServiceProvider" --tag=icd10

# Run to migrate the ICD-10 table
php artisan migrate

# Run to seed the data
php artisan db:seed --class=Icd10Seeder
```

