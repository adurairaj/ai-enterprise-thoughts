#Check list for record deletion
1. Delete only if primary key is present
2. Check if timestamp validation is required
3. Check if there are any dependant records on the row to be deleted.
4. Apply validations
5. if records should be soft deleted, then mark the record as deleted
6. if the record should be hard deleted, then delete the record
