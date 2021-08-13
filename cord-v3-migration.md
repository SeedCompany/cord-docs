# Cord V2 – V3 Migration


### Projects
- `Project.mouEnd` in V3 is taken from `Project.estCmpl` in V2, which is an array of estimated completion dates (mou ends) along with `Project.mouEnd` in V2. The way this was migrated is that for every value in `estCmpl` the migrated V3 project's mou end was updated. The last element of this array should, but does not always match the current mou end. If the last element did not equal the current mou end, then the migrated project was updated again with that value. 

### Engagements
- In V3 `Engagement.modifiedAt` is taken from V2 `Engagement.createdAt` as an approximation since that data was not tracked in V2.

### Locations
- There is a `Location` with a name of "Not Applicable", a `FundingAccount` with a name of "Not Specified", 
a `FieldZone` with a name of "Not Speicifed" and a `FieldRegion` with a name of "any". I believe these represented obfuscated locations in V2. 
There are a number of `Project`s pointing to this `Location` and the related "Not Specified"/"any" location-adjacent resources.
I believe the primary location for these projects could be replaced with `null`. All of these un-specified location-adjacent resources could be removed from the db.

### Partners
- Every migrated partner was assigned `Funded` and `FieldEngaged` for their `financialReportingTypes` per Seth's instructions. 

### Status History
- For Projects and Engagements, if they had `statusHistory` from V2, then after initial migration the project/engagement had its step/status updated for each entry in `statusHistory`.
This part of the migration was not handled well (mea culpa) and will need to be fixed if the API ever exposes status history. Here is the before and after:

![Screen Shot 2021-08-09 at 4 55 19 PM](https://user-images.githubusercontent.com/49688912/128779542-a73e2f73-d21b-4ece-9f9b-a66709f2b7dc.png)

![Screen Shot 2021-08-09 at 6 07 36 PM](https://user-images.githubusercontent.com/49688912/128785271-c1004f30-c616-4e0a-8511-31d193e24d1d.png)

The first step in Neo4j (formerly status in Mongo) is "EarlyConversations". This property node (Neo4j) also includes the timestamp when the step was updated 
_to the next_ step and the user that updated the step. Thus the current step has no timestamp or user. The timestamp of "PrepForConsultantEndorsement" was when the step
became "PendingConceptApproval". Essentially each step node is one behind in the sequence and it's a bit confusing. 
What should happen here is that the timestamp for each project.step node should be applied to the the `createdAt` 
for the next step `(:Project)-[:step { createdAt: $timestampFromPreviousStep } ]->(:Property { createdAt: $timestampFromPreviousStep })`. 
So, in Neo4j `"EarlyConversations".timestamp` should become `"PendingConceptApproval".createdAt` and be removed from "EarlyConversations".
In order to find the order of steps, you can order by `timestamp` and apply the `timestamp` to the next step node.
As far as the user who updated the step, we don't currently track this in V3, so this can't be migrated too well at the moment. Everything said about project.step should be applied to engagement.status as well.

### CreatedAt
- Migrated `BaseNode`s from V2 have their `createdAt` property taken from V2. In the case of Projects, this was V2 `project.createDate`. 
If that was missing from the project, and for all other base nodes, the Mongo ObjectID creation timestamp was used. In V3, relationships and properties have
`createdAt` timestamps. The `BaseNode` `createdAt`s were never spread to the related relationships and properties (with the exception of project member nodes and the direct relationships from the project to the member nodes). The issue is [here](https://github.com/SeedCompany/seed-api/issues/25).
The tricky bit with this is that you can't blindly propagate/spread the `createdAt` property indefinitely. This is because other `BaseNode`s will be encountered 
that have their own `createdAt` (for instance, a Project is related to a Location, but both have their own `createdAt`). 
Another case is that you can't apply Engagement.createdAt to the `ProgressReport` `createdAt`s since these were added after the engagement was created. 
Propagation of `createdAt` will need to be done on a case by case basis for each `BaseNode`/resource. 
It might also be the case that this will prove very difficult to accomplish since there have been many property updates since the time of migration (2/13/2021). The `createdAt` for an updated property and property relationship will not match the `createdAt` of the `BaseNode`.

### Migration Discrepancies
- [Fixed & in need of research](https://seedcompany.sharepoint.com/:x:/r/sites/DataAnalytics/_layouts/15/guestaccess.aspx?e=4%3AkmlAdZ&at=9&wdLOR=c876101D7-85B8-5543-A3E5-10027F73BD28&share=EXEyWJ51idlBqfWKOSogwYgBt3sUnr3cbcn7Mfqh7-Nv2w)

### Migration Script
- To see what happened in code, you can checkout commit 60f598f in `seed-api` and look at `cord-migration-service.ts`

