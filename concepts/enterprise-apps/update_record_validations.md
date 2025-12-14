# Check list for Update record validations
1. If a primary key is defined, then verify if the primary key already exists in the database
   1. if yes, proceed with updation.
   2. if no, throw error saying "ERROR: Record does not exist"
2. if value for non null fields are not provided, then throw error saying "ERROR: <field name> cannot be empty"
3. Validate each field's value.
4. Save the record to database
