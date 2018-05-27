# kshift

Kshift is a ETL tool to persist data from kafka to Any Store. As of now it supports Redshift and S3 (Redshift Spectrum) as Sinks.

Below is the explaination of various configs:

| Config Name	| Description	Type	| Mandatory |	Default|
|---|----|---|--|
isUpsert	| If true, data will be upserted. If false, data will be appended. Key on kafka queue is mandatory in this case as data is deduplicated using key only.	| Boolean	| No	| false 
