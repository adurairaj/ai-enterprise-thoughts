# This file contains validations for  inserting a new record.
1. Verify if the primary key is an auto number
  1. if yes then do not check for duplicate primary key.
  2. if not verify if a record with that primary key already exists. if it exists throw an error "ERROR: RECORD already exists"
2. Verify if all the non null values are populated.
3. 
