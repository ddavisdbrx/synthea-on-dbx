# synthea-on-dbx
### Generate Synthetic Health Care Records using Synthea Directly On Databricks

Synthetic Health's [Synthea Patient Generator](https://github.com/synthetichealth/synthea) is a great resource for generating realistic looking synthetic health care data in a variety of formats including FHIR, C-CDA, CPCDS and a light weight, easy to use CSV format.  

The data includes encounters, conditions, allergies, care plans and more!  

Synthea can be executed by issueing commands to a downloadable JAR file that requires the Java JDK version 11 or higher.  Historically Databricks users would execute Synthea on a local machine to generate some files and then upload them to cloud storage for use.  

The purpose of this repository is to allow a Databricks user to generate synthetic patient records using Synthea directly using a Databricks Workflow.  The workflow can be executed indepently once its created, or it can be looped with a random wait time to help simulate the variablity in patient records flowing into a health system.  This is particularly useful for demostrating streaming ETL methods such as for Spark Structured Streaming or Delta Live Tables.  

***

#### Usage:  

1. Clone this repo into a git controlled folder in your Databricks **Unity Catalog** enabled workspace of choice. The user must either be allowed to create a **catalog**, have full use permissions on an existing catalog to create a **schema**, or have permissions in a catalog.schema to use and create a **volume**.
1. Open the ***synthea-on-databricks*** notebook.
1. Attach the notebook to your Databricks Interactive Cluster of choice.  The notebook has been tested with both Databricks Generic Serverless compute and with a Single Node Assigned User cluster running DBR 14.3 LTS.  
1. Run the first few commands of the notebook to install or upgrade the Databricks Python SDK and set the required values for the Databricks Widgets:   
  a. **catalog_name**: the catalog that will be created and/or used for the volume that will be created. Note that there is no default for this value and must be set by the user.  
  b. **schema_name**: the schema that will be created and/or used for the volume that will be created.  The default for the schema name is "synthea".  Note that if more than one user wishes to run this notebook that either **catalog_name** or **schema_name** must be unique for a unique volume to be created.  It's also OK for both users to write to the same volume as long as they both have permissions to use it.  
  c. **node_type_id**: If an **instance_pool_id** is not set, the **node_type_id** specifies which type of compute resouce will be used to create the assigned single node cluster used to create and execute the workflow.  This defaults to *i3.xlarge*, which is an AWS **node_type_id**.  The Databricks Python SDK Workspace Client may be used to list the available node types in your workspace with `nodes = w.clusters.list_node_types()`.  Please review the Databricks Python SDK [documentation](https://docs.databricks.com/en/dev-tools/sdk-python.html) for more information.  
  d. **instance_pool_id**: If your workspace has a pool of clusters available to be used for job compute, you may optionally specify the **instance_pool_id**.  These ids are specific to a Databricks workspace and can be found using the Databricks Python SDK Workspace Client with `pools = w.instance_pools.list()`.
  e. **number_of_job_runs**:  Once the Databricks Workflow has been posted to the Workspace, you can optionally loop its execution a number of times using this input variable.  The default is one time.  Interactively executing the final commands in the notebook will also execute the Workflow with `w.jobs.run_now_and_wait(...)`.  This variable is used for convenience when demonstrating later streaming workflows that will utilize the synthetic data generated.  
1. Run the rest of the ***synthea-on-databricks*** notebook to peform the following actions:    
  a. Post a conditional Databricks Workflow to the Workspace.  The workflow will check to see if Synthea's JAR and configuration file is available in the desired **volume** in Unity Catalog.  If its there, the workflow will immediately begin running the data generator which will generate a random number patient records between 1 and 1000 records.  If the JAR and configuration file are not available, the workflow will create or replace the catalog, schema, and volume required, download the latest JAR from the Synthea github page, and write the configuration file that will be used for its execution.  
  b. Excute the workflow from the notebook a given number of times as set above.   

***

#### Limitations: 

The first release of this workflow generates between 1 and 1000 patient records in a collection of 18 CSV files that are saved in the output/csv subfolder of the volume.  The patient records are also all from the default geographic location: Boston, MA.  Synthea is capable of taking several options including the generation different file types including comprehensive HL7 FHIR for patients in different geographic locations.  The notebook `notebooks/00-setup-notebooks/0.3-synthea-configuration` is capable of taking several Databricks Widgets inputs to control the execution of the Synthea JAR but the workflow has not been set up yet to accpet this additional values.  Adjusting the defaults used in the notebook manually in your cloned copy of this repo will allow for setting other runtime settings.  Future releases of this repo will continue the additional options required in the **synthea-on-databricks** notebook.  




















  