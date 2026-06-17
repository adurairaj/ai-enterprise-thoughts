The software developed should be queryable.
e.g for adding a new customer how many times db is hit? What are the queries fired?
Can an unauthorized user create a customer?

With LLM based code generators on the rise, this feature is of great importance, because it is very difficult for people to find bugs in the generated code.
It is simply overwhelming for the humans to make sure the code generated is safe and compliant.

For this to happen, generating code in programming languages like Java,Python, C/C++ will not help.
Instead we need to generate code in some DSL, where it is easy to verify what is going on.

Can you answer the following questions without digging into your code?
1. which REST APIs access customer table?
2. which REST APIs access age of the customer?
3. What validations are specified on customer entity?
4. How did this claim got processed?
5. How many queries join more than 2 tables?
6. What service class should I use to fetch customer.address?
7. In a given REST API how many times database is hit?
8. What REST APIs are used in this screen?
