# 1. Manually Create Token

```php
/** 
 * Uji coba echo Token yang sesuai dan di DB akan tersimpan
 * Pastikan sudah mengisi konfigurasi di .env
*/

$client = new Satusehat\Integration\OAuth2Client;
echo $client->token(); // OAuth2Token anda akan muncul
```

# 2. KYC (Verifikasi SATUSEHAT Centang Biru)
*Note : Wajib dilakukan pada konfigurasi .env ```SATUEHAT_ENV=PROD```

```php
/** 
 * Pastikan sudah mengisi konfigurasi di .env
 * Proses KYC tidak perlu lagi menggunakan deklarasi OAuth2Client->token()
*/
<?php

use Satusehat\Integration\KYC;

$kyc = new KYC;

// Isi nama verifikator & NIK verifikator untuk mendapatkan link KYC
$json = $kyc->generateUrl('{nama_verifikator}', '{nik_verifikator}');
$kyc_link = json_decode($json, true);

/** 
 * Melakukan route redirect ke link KYC
 * saat ini hanya bisa dibuka pada tab baru / pop-up
 * tidak bisa melalui iframe
*/
return redirect($kyc_link['data']['url']);
```

# 3. FHIR Resource Agnostic Operation
- Fungsi dasar GET, POST, PUT sudah ditambahkan pada kelas OAuth2Client.
- Proses GET / POST / PUT, tidak perlu lagi menggunakan deklarasi ```OAuth2Client->token()```
- Hasil balikan dari GET adalah kode status HTTP dan response body (JSON)

## Contoh GET by ID

```php
/** 
 * Hasil balikan dari GET / POST / PUT adalah kode status HTTP dan response body (JSON)
*/
<?php

use Satusehat\Integration\OAuth2Client;

$client = new OAuth2Client;

// Organization
[$statusCode, $response] = $client->get_by_id('Organization', '{id}');

// Location
[$statusCode, $response] = $client->get_by_id('Location', '{id}');

// Patient
[$statusCode, $response] = $client->get_by_id('Patient', '{id}');

// Practitioner
[$statusCode, $response] = $client->get_by_id('Practitioner', '{id}');

// Encounter
[$statusCode, $response] = $client->get_by_id('Encounter', '{id}');

// Condition
[$statusCode, $response] = $client->get_by_id('Condition', '{id}');
```

## GET by NIK

```php
// Patient
[$statusCode, $response] = $client->get_by_nik('Patient', '{NIK Pasien}');

// Practitioner
[$statusCode, $response] = $client->get_by_nik('Practitioner', '{NIK Dokter}');
```

## Agnostic POST & PUT : using self build JSON object

```php
$client = new OAuth2Client;
$body = ...... ; // JSON Object
$resource = ......; // FHIR Resource (Bundle, Organization, Location, Patient, Practitioner, Encounter, Condition)

// Format Agnostic POST
// All resources accepted
[$statusCode, $response] = $client->ss_post($resource, $body);
echo $statusCode, $response;

// Format Agnostic PUT
// All resources except Bundle
$id = ...... ; // SATUSEHAT response ID
[$statusCode, $response] = $client->ss_put($resource, $id, $body);
echo $statusCode, $response;
```

# 4. Menggunakan FHIR Class Object
Untuk mempermudah pembuatan FHIR Object data type, dapat menggunakan class object yang sudah disediakan.<br>
Pertama-tama import class dari FHIR Class Object :
```php
<?php

use Satusehat\Integration\FHIR\Organization;
use Satusehat\Integration\FHIR\Location;
use Satusehat\Integration\FHIR\Patient;
use Satusehat\Integration\FHIR\Encounter;
use Satusehat\Integration\FHIR\Condition;
use Satusehat\Integration\FHIR\Bundle;
```

## Organization
```php
// Organization
$organization = new Organization;
$organization->addIdentifier('{kode_unik_suborganisasi}'); // unique string free text (increments / UUID / inisial)
$organization->setName('{nama_suborganisasi}'); // string free text
$organization->json();
```

## Location
```php
// Location
$location = new Location;
$location->addIdentifier('{kode_unik_lokasi}'); // unique string free text (increments / UUID / inisial)
$location->setName('{nama_lokasi}'); // string free text
$location->addPhysicalType('{tipe_lokasi}'); // ro = ruangan, bu = bangunan, wi = sayap gedung, ve = kendaraan, ho = rumah, ca = kabined, rd = jalan, area = area. Default bila tidak dideklarasikan = ruangan
$location->json();
```

## Patient
```php
// Patient
$patient = new Patient;
$patient->addIdentifier('{nik/nik-ibu}', '{nomor_nik}');
$patient->setName('{nama_pasien}');

/*
 *  Defaultnya adalah nomor telepon. Kalau ingin yang lain bisa seperti
 *  $patient->addTelecom('{telecom_value}', '{telecom_system}', '{telecom_use}')
 *  Telecom system : https://www.hl7.org/fhir/R4/valueset-contact-point-system.html
 *  Telecom use : https://www.hl7.org/fhir/R4/valueset-contact-point-use.html
*/
$patient->addTelecom('{nomor_telepon}');

$address_detail = [
    'address' => '{alamat}',
    'city' => '{kota}',
    'postalCode' => '{kode_pos}',
    'country' => 'id-ID', // Kode negara
    'provinceCode' => '{kode_provinsi}',
    'cityCode' => '{kode_kabupaten/kota}',
    'districtCode' => '{kode_kecamatan}',
    'villageCode' => '{kode_wilayah}',
    'rt' => '{rt}',
    'rw' => '{rw}',
];

$patient->setGender('{male/female}');
$patient->setBirthDate('{YYYY-MM-DD}');
$patient->setDeceased('{boolean}');
$patient->setAddress($address_detail);

$patient->setMaritalStatus('{marital_status}');  // Married, unmarried, never, divorced, widowed

$patient->setMultipleBirth('{integer}'); // menunjukkan urutan kelahiran yang sebenarnya

$patient->setEmergencyContact('{nama_kontak}', '{nomor_kontak}');
$patient->setCommunication(); // Bahasa pasien, default Indonesian

$patient->json();
```

## Encounter
```php
// Encounter
$encounter = new Encounter;

/** 
 * timestamp_kedatangan, timestamp_mulai/akhir_pemeriksaan, timestamp_pulang berupa datetime dan nanti akan dikonversi menjadi ATOM format (Y-m-dTH:i:s+UTC)
 * Contoh : 31 Desember 2022 15:45 WIB : 2022-12-31T15:45:00+07:00
*/

$encounter->addRegistrationId('{kode_registrasi}'); // unique string free text (increments / UUID)

$encounter->setArrived('{timestamp_kedatangan}');
$encounter->setInProgress('{timestamp_mulai_pemeriksaan}', '{timestamp_akhir_pemeriksaan}');
$encounter->setFinished('{timestamp_pulang}');

$encounter->setConsultationMethod('{metode_konsultasi}'); // RAJAL, IGD, RANAP, HOMECARE, TELEKONSULTASI
$encounter->setSubject('{id_patient}', '{nama_pasien}'); // ID SATUSEHAT Pasien dan Nama SATUSEHAT
$encounter->addParticipant('{id_practitioner}', '{nama_dokter}'); // ID SATUSEHAT Dokter, Nama Dokter
$encounter->addLocation('{id_location}', '{nama_poli}'); // ID SATUSEHAT Location, Nama Poli
$encounter->addDiagnosis('{id_condition}', '{kode_icd_10}'); // ID SATUSEHAT Condition, Kode ICD10
$encounter->json();
```

## Condition
```php
// Condition
$condition = new Condition;
$condition->addClinicalStatus('{status_klinis}'); // active, inactive, resolved. Default bila tidak dideklarasi = active
$condition->addCategory('{kategori}'); // Diagnosis, Keluhan. Default : Diagnosis
$condition->addCode('{kode_icd_10}'); // Kode ICD10
$condition->setSubject('{id_patient}', '{nama_pasien}'); // ID SATUSEHAT Pasien dan Nama SATUSEHAT
$condition->setEncounter('{id_encounter}'); // ID SATUSEHAT Encounter
$condition->setOnsetDateTime('{timestamp_onset}'); // timestamp onset. Timestamp sekarang
$condition->setRecordedDate('{timestamp_recorded}'); // timestamp recorded. Timestamp sekarang
$condition->json();

```

## Bundle (Encounter & Condition)
```php
// Buat dulu Encounternya
$encounter = new Encounter;
$encounter->addRegistrationId('{kode_registrasi}'); // unique string free text (increments / UUID)
$encounter->setArrived('{timestamp_kedatangan}');
$encounter->setInProgress('{timestamp_mulai_pemeriksaan}', '{timestamp_akhir_pemeriksaan}');
$encounter->setFinished('{timestamp_pulang}');
$encounter->setConsultationMethod('{metode_konsultasi}'); // RAJAL, IGD, RANAP, HOMECARE, TELEKONSULTASI
$encounter->setSubject('{id_patient}', '{nama_pasien}'); // ID SATUSEHAT Pasien dan Nama SATUSEHAT
$encounter->addParticipant('{id_practitioner}', '{nama_dokter}'); // ID SATUSEHAT Dokter, Nama Dokter
$encounter->addLocation('{id_location}', '{nama_poli}'); // ID SATUSEHAT Location, Nama Poli

// Buat Condition
$condition1 = new Condition;
$condition1->addClinicalStatus(); // active, inactive, resolved. Default bila tidak dideklarasi = active
$condition1->addCategory('Diagnosis'); // Diagnosis, Keluhan. Default : Diagnosis
$condition1->addCode('{kode_icd10}'); // Kode ICD10
$condition1->setSubject('{id_pasien}', '{nama_pasien}'); // ID SATUSEHAT Pasien dan Nama SATUSEHAT
$condition1->setOnsetDateTime({timestamp_onset}); // timestamp onset. Timestamp sekarang
$condition1->setRecordedDate({timestamp_recorded}); // timestamp recorded. Timestamp sekarang
$condition1->json();

// Jika terdapat 2 condition
$condition2 = new Condition;
// Data pendukung condition

// Setelah Encounter dan Condition dibuat, kita buat bundlenya
$bundle = new Bundle;
$bundle->addEncounter($encounter)
$bundle->addCondition($condition1)
$bundle->addCondition($condition2) // Jika terdapat 1 atau lebih condition
```

# 5. POST / PUT menggunakan FHIR Class Object
Proses GET / POST / PUT melalui FHIR class Object, tidak perlu lagi menggunakan deklarasi kelas OAuth2Client.<br>
Hasil balikan dari POST & PUT adalah kode status HTTP dan response body (JSON)

```php
// Organization
[$statusCode, $response] = $organization->post();
[$statusCode, $response] = $organization->put('{organization_id}');

// Location
[$statusCode, $response] = $location->post();
[$statusCode, $response] = $location->put('{location_id}');

// Patient
[$statusCode, $response] = $patient->post();

// Encounter
[$statusCode, $response] = $encounter->post();
[$statusCode, $response] = $encounter->put('{encounter_id}');

// Condition
[$statusCode, $response] = $condition->post();
[$statusCode, $response] = $condition->put('{condition_id}');

// Bundle
[$statusCode, $response] = $bundle->post();
```
