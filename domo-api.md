# Domo API

### Summary
This API (code in `seed-api`) maps data from the Cord V3 API to Domo datasets that Data Analytics uses for the organization.

### Env vars for local testing
You need to test and audit new datasets. Coordinate with Ken Jones and get his sign-off before merging into production. 
Make sure `DOMO_DATASET_ENV` is set to `test`.
```
DOMO_URL=https://api.domo.com
DOMO_CLIENT_ID=****... (can be found in AWS secrets manager)
DOMO_CLIENT_SECRET=****... (can be found in AWS secrets manager)
DOMO_DATASET_ENV=test 
```

### Domo Sync Command
This runs the sync for a given Domo API. Multiple can be specified. 
`yarn console c2d sync $nameOfDomoApi`
If a previous sync has been run and the schema is unchanged, you can run the above with the `--data` flag to skip the schema sync step. 

### Domo API Schema
Schema definitions for the various Domo datasets are in `components/domo/datasets/cord`

### Domo API Syncs
The files containing the mapping between the Cord API and Domo
in `components/cord-to-domo/{resourceName}`

### DataSets in Domo UI
- https://seedcompany.domo.com/datacenter/datasources
- Ken has to share new DataSets for you to see them here, but they should look like: `CF3.{DataSetName}.TEST`. Production datasets are `CF3.{DataSetName}.PROD`.
