# Check list for Update record validations
1. If a primary key is defined, then verify if the primary key already exists in the database
   1. if yes, proceed with updation.
   2. if no, throw error saying "ERROR: Record does not exist"
2. verify if the timestamp of reading the record is still the same in the database.
  1. if they are not same , throw error saying "record has been modified since you have seen it, load the latest changes"
2. if value for non null fields are not provided, then throw error saying "ERROR: <field name> cannot be empty"
3. Validate each field's value.
4. Save the record to database
