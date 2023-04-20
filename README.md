
A simulated music streaming App has grown their user base and song database and wants to move their processes and data onto the cloud. Their data resides in S3, in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

This project builds a pipeline by extracting data from S3 and load them onto Redhsift. A star Schema is designed with a fact table songplay and four dimensional tables for further business intelligence.
