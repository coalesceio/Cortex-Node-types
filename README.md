# Cortex Package

* [ML Forecast](#ml-forecast)
* [ML Anomaly Detection](#ml-anomaly-detection)
* [LLM Cortex Functions](#llm-cortex-functions)
* [Top Insights](#top-insights)
* [Classification](#classification)
* [Document AI](#document-ai)
* [Cortex Search Service](#cortex-search-service)
* [Code](#code)
---

## ML Forecast

The Coalesce ML Forecast UDN is a versatile node that allows you to create a forecast table and insert forecasts of time series data using the Snowflake built-in class [FORECAST](https://docs.snowflake.com/sql-reference/classes/forecast).

[Snowflake Cortex](https://docs.snowflake.com/en/user-guide/ml-powered-forecasting) is Snowflake's intelligent, fully-managed service that enables organizations to quickly analyze data and build AI applications, all within Snowflake. This service makes Machine Learning (ML) functionality accessible to data engineers to enrich data pipelines while still using SQL. Forecasting employs a machine learning algorithm to predict future data by using historical time series data.

### Node Configuration

The ML Forecast has two configuration groups:

* [Node Properties](#node-properties)
* [Forecast Model Input](#forecast-model-input)

#### Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the Forecast table will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### Forecast Model Input

| **Option** | **Description** |
|------------|----------------|
| **Model Instance Name** | (Required) Name of the model that needs to be created |
| **Create Model** | True/False toggle to determine model creation:<br/>- **True**: Forcefully create Forecast model<br/>-- **Series Column** (required for multi-series): For multiple time series models, the name of the column defining the multiple time series in input data.<br/>- **False**: Refer to existing Forecast model |
| **Multi-Series Forecast** | True/False toggle for forecast type:<br/>- **True**: Create multi-series forecast model with series column, timestamp column and target column<br/>- **False**: Specify the timestamp column and target column to create single-series forecast model |
| **Series Column** | (Required for multi-series) Column defining multiple time series in input data |
| **Timestamp Column** | (Required) Column containing timestamps in input data |
| **Target Column** | (Required) Column containing target values in input data | 
| **Config object** | OBJECT containing key-value pairs to configure forecast job |
| **Series value** | Required for multi-series forecasts. Single value or VARIANT |
| **Exogenous Variables** | True/False toggle:<br/>- **True**: Add future-valued exogenous data using multi-source toggle<br/>- **False**: Create forecast model based on days to forecast only |
| **Multi Source** | Toggle to add future-valued Exogenous data |
| **Days to Forecast** | (Required for forecasts without exogenous variables) Number of steps ahead to forecast |

### ML Forecast Preprocessing of Data

When the forecast model returns an error, the error message returned by Snowflake is captured and surfaced directly in the Coalesce application for troubleshooting.

Common scenarios you may encounter:

* NULLS in the source data. The model will cope with some, but not too many NULLS.
* Missing time periods. If the model is unable to determine a consistent frequency in the time series it will cause an error.
* Missing exogenous variables. If the model was trained with exogenous variables.
* Exogenous variables need to be provided into the future to predict future values.

A data preparation step in Coalesce can be used prior to the ML Forecast node to address these issues.

### ML Forecast Deployment

#### ML Forecast Initial Deployment

When deployed for the first time into an environment the ML Forecast node will execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Forecast Table** | This will execute a CREATE OR REPLACE statement and create a Forecast Table in the target environment |

#### ML Forecast Redeployment

After the ML Forecast node has been deployed for the first time into a target environment, subsequent deployments may result in altering the forecast table.

#### ML Forecast Altering the Forecast Tables

The following column or table changes that is made in isolation or all-together will result in an ALTER statement to modify the Forecast table in the target environment:

* Change in table name
* Dropping existing column  
* Alter column data type
* Adding a new column

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Clone Table** | Creates an internal table |
| **Rename Table/Alter Column/Delete Column/Add Column/Edit table description** | Alter table statement is executed to perform the alter operation accordingly |
| **Swap cloned Table** | Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost |
| **Delete Table** | Drops the internal table |

### ML Forecast Undeployment

If a ML Forecast table is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Forecast Table in the target environment will be dropped.

This is executed in two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete Table** | Coalesce Internal table is dropped |
| **Delete Table** | Target table in Snowflake is dropped |

---

## ML Anomaly Detection

The Coalesce ML Anomaly Detection UDN is a versatile node that allows you to create an Anomaly table and insert anomalies of time series data using the Snowflake built-in class [ANOMALY DETECTION](https://docs.snowflake.com/en/sql-reference/classes/anomaly_detection).

[Snowflake Cortex](https://docs.snowflake.com/en/user-guide/ml-powered-anomaly-detection) is Snowflake's intelligent, fully-managed service that enables organizations to quickly analyze data and build AI applications, all within Snowflake. This service makes Machine Learning (ML) functionality accessible to data engineers to enrich data pipelines while still using SQL. Anomalies in data are detected by analyzing the dataset using a machine learning algorithm.

### ML Anomaly Detection Node Configuration

The ML Anomaly has two configuration groups:

* [Node Properties](#ml-anomaly-node-properties)
* [Anomaly Model Input](#ml-anomaly-model-input)

#### ML Anomaly Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the Table will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### ML Anomaly Model Input

| **Option** | **Description** |
|------------|----------------|
| **Model Instance Name** | (Required) Name of the model that needs to be created |
| **Create Model** | True/False toggle to determine if model can be created or refer to an existing model:<br/>- **True**: Forcefully create Anomaly model<br/>- **False**: Refer to existing Anomaly model |
| **Multi-Series** | True/False toggle for series type:<br/>- **True**: Create multi-series model with series column, timestamp column and target column<br/>- **False**: Create single-series model |
| **Series Column** | (Required for multi-series) Column defining multiple time series in input data |
| **Timestamp Column** | (Required) Column containing timestamps in input data |
| **Target Column** | (Required) Column containing target(dependent) values in input data |
| **Config object** | Object containing configuration settings for anomaly detection job |
| **Supervised Data** | Toggle to train model using labeled data through multi-source toggle |
| **Labeled Column** |(Availbe with supervised datas) Essential for supervised data, distinguishes between normal and anomalous instances |
| **Unsupervised Data** | Toggle to train model using historical data through multi-source toggle |
| **Multi Source** | Toggle to add data for analysis |

### ML Anomaly Detection Preprocessing of Data

When the Anomaly model returns an error, the error message returned by Snowflake is captured and surfaced directly in Coalesce for troubleshooting.

Common scenarios you may encounter:

* Missing time periods. If the model is unable to determine a consistent frequency in the time series it will cause an error.
* Missing labeled column. If the model was trained with supervised data it's necessary to pass a labeled column.

Often, a data preparation step in Coalesce can be used prior to the ML Anomaly Detection node to address these issues.

### ML Anomaly Detection Deployment

#### ML Anomaly Detection Initial Deployment

When deployed for the first time into an environment the ML Anomaly Detection node will execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Anomaly Detection Table** | This will execute a CREATE OR REPLACE statement and create an Anomaly Table in the target environment |

#### ML Anomaly Detection Redeployment

After the ML Anomaly node has been deployed for the first time into a target environment, subsequent deployments may result in altering the Anomaly table.

#### ML Anomaly Detection Altering the Anomaly Tables

The following column or table changes that is made in isolation or all-together will result in an ALTER statement:

* Change in table name
* Dropping existing column
* Alter column data type
* Adding a new column

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Clone Table** | Creates an internal table |
| **Rename Table/Alter Column/Delete Column/Add Column/Edit table description** | Alter table statement executed to perform operation |
| **Swap cloned Table** | Clone replaces main table after successful updates |
| **Delete Table** | Drops the internal table |

### ML Anomaly Detection Undeployment

If a ML Anomaly table is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Anomaly Table in the target environment will be dropped.

This is executed in two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete Table** | Coalesce Internal table is dropped |
| **Delete Table** | Target table in Snowflake is dropped |

---

## LLM Cortex Functions

The Coalesce Cortex Function UDN provides instant access to industry-leading large language models (LLMs) developed by researchers. Additionally, it offers models that Snowflake has finely tuned for specific use cases.

[Snowflake Cortex - LLM Functions](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions) LLMs are fully hosted and managed by Snowflake, using them requires no setup. Your data stays within Snowflake, giving you the performance, scalability, and governance you expect.

### LLM Cortex Functions Node Configuration

The LLMs Cortex function has three configuration groups:

* [Node Properties](#llm-cortex-functions-node-properties)
* [Options](#llm-cortex-functions-options)
* [Cortex Package](#llm-cortex-functions-cortex-package)

#### LLM Cortex Functions Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the Table will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### LLM Cortex Functions Options

| **Option** | **Description** |
|------------|----------------|
| **Create As** | Provides option to choose materialization type as `table` |
| **Multi Source** | True/False toggle for SQL UNIONs:<br/>- **True**: Multiple sources combined using Multi Source Strategy<br/>- **False**: Single source or join |
| **Truncate Before** | True/False toggle:<br/>- **True**: INSERT OVERWRITE used<br/>- False: INSERT used to append data |
| **Enable tests** | Toggle to enable testing features |
| **Pre-SQL** | SQL to execute before data insert operation |
| **Post-SQL** | SQL to execute after data insert operation |

#### LLM Cortex Functions Cortex Package

| **Option** | **Description** |
|------------|----------------|
| **SUMMARIZE** | True/False toggle if the data from the column should be returned as a summary:<br/>- **True**: System prompts to add a column<br/>- **False**: Function remains inactive |
| **SENTIMENT** | True/False toggle to return sentiment score (-1 to 1) for English text, with -1 being the most negative, 0 is neutral, and 1 is positive.:<br/>- **True**: System prompts to add a column<br/>- **False**: Function remains inactive |
| **TRANSLATE** | True/False toggle to translate text:<br/>- True: System prompts to add a column<br/>- False: Function remains inactive |
| **EXTRACT ANSWER** | True/False toggle to extract answers:<br/>- **True**: System prompts to add a column<br/>- **False**: Function remains inactive |

### LLM Cortex Functions Key Points for Consideration

1. Review the [required privileges](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions)
2. Datatype of target column for using function "Extract Answer" should be an Array
3. If source data is required in the target column, duplication of column is necessary before applying a cortex function

### LLM Cortex Functions Deployment

#### LLM Cortex Functions Initial Deployment

When deployed for the first time into an environment the LLM node will execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Table** | This will execute a CREATE OR REPLACE statement and create a Table in the target environment |

#### LLM Cortex Functions Redeployment

After the LLM node has been deployed for the first time into a target environment, subsequent deployments may result in altering the table.

#### Altering the LLM Cortex Functions Tables

The following column or table changes if made in isolation or all-together will result in an ALTER statement:

* Change in table name
* Dropping existing column
* Alter Column data type  
* Adding a new column

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Clone Table** | Creates an internal table |
| **Rename Table/Alter Column/Delete Column/Add Column/Edit table description** | Alter table statement executed to perform operation |
| **Swap cloned Table** | Clone replaces main table after successful updates |
| **Delete Table** | Drops the internal table |

### LLM Cortex Functions Undeployment

If a LLM Node is deleted from a Workspace, and that Workspace is committed to Git, subsequently deployed to a higher-level environment, then the table in the target environment will be dropped.

This is executed in two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete Table** | Coalesce Internal table is dropped |
| **Delete Table** | Target table in Snowflake is dropped |

---

## Top Insights

The Coalesce Top Insights UDN is a versatile node that allows you to streamline and improve the process of root cause analysis around changes in observed metrics. Learn more about [Top Insights](https://docs.snowflake.com/en/user-guide/snowflake-cortex/ml-functions/contribution-explorer).

### Top Insights Node Configuration

The Top Insights node has two configuration groups:

* [Node Properties](#ml-contribution-explore-node-properties)
* [Top Insights](#ml-contribution-explorer-config)

#### Top Insights Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the Table will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### Top Insights Configuration

| **Option** | **Description** |
|------------|----------------|
| **Create As** | Provides option to create a 'view' |
| **CATEGORICAL DIMENSIONS** | (Required) Categorical attributes essential for analysis |
| **CONTINUOUS DIMENSIONS** | (Required) Columns representing continuous aspects that vary within a range |
| **Metric** | (Required) Column representing target metric being investigated |
| **Label** | (Required) Column distinguishing between control (FALSE) and test (TRUE) data |
| **Filter Insights** | Textbox to customize filtering of top insights |
| **Order By** | True/False toggle:<br/>- **True**: Sort column and sort order visible and required for order by clause<br/>- **False**: Sort column and sort order invisible |

### Top Insights Deployment

#### Top Insights Initial Deployment

When deployed for the first time into an environment the View node will execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Create or replace View** | This will execute a CREATE OR REPLACE statement and create a View in the target environment |

#### Top Insights Redeployment

The subsequent deployment of a View node with changes in the view definition, altering options, or renaming the view results in the deletion of the existing view and the recreation of the view.

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete View** | Removes existing view |
| **Create View** | Creates new view with updated definition |

### Top Insights Undeployment

If a View Node is removed from a Workspace, and the changes are committed to Git and deployed to a higher-level environment, the corresponding View in the target environment will be dropped.

This is executed in a single stage:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete View** | Drops the existing View from target environment |

---

## Classification

The Coalesce Classification is a versatile node that allows you to create a classification table and classification model to classify data into different classes using patterns detected in training data using in-built snowflake ML function [CLASSIFICATION](https://docs.snowflake.com/en/sql-reference/classes/classification).

Classification uses machine learning algorithms to sort data into different classes using patterns detected in training data. Binary classification (two classes) and multi-class classification (more than two classes) are supported. Common use cases of classification include customer churn prediction, credit card fraud detection, and spam detection.

### Classification Node Configuration

The Classification node has two configuration groups:

* [Node Properties](#classification-node-properties)
* [Classification Model Input](#classification-model-input)

#### Classification Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the Table will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### Classification Model Input

| **Option** | **Description** |
|------------|----------------|
| **Model Instance Name** | (Required) Name of the model that needs to be created |
| **Create Model** | True/False toggle to determine model creation:<br/>- **True**: Forcefully create Classification model<br/>- **False**: Refer to existing Classification model |
| **Target Column** | (Required) Column containing target values in input data |
| **Multi Source** | Toggle to add data for analysis |

### Classification Deployment

#### Classification Initial Deployment

When deployed for the first time into an environment the Classification node will execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Classification Table** | This will execute a CREATE OR REPLACE statement and create a Classification Table in the target environment |

#### Classification Redeployment

After the Classification node has been deployed for the first time into a target environment, subsequent deployments may result in altering the Classification table.

#### Altering the Classification Tables

The following column or table changes that is made in isolation or all-together will result in an ALTER statement:

* Change in table name
* Dropping existing column
* Alter column data type
* Adding a new column

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Clone Table** | Creates an internal table |
| **Rename Table/Alter Column/Delete Column/Add Column/Edit table description** | Alter table statement executed to perform operation |
| **Swap cloned Table** | Clone replaces main table after successful updates |
| **Delete Table** | Drops the internal table |

### Classification Undeployment

If a Classification table is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Classification Table in the target environment will be dropped.

This is executed in two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete Table** | Coalesce Internal table is dropped |
| **Delete Table** | Target table in Snowflake is dropped |

## Document AI

The Coalesce Document AI UDN is a node that allows you to develop and deploy a processing pipeline using the already prepared Document AI model build, streams, and tasks. The pipeline will extract information from new inspection documents stored in an internal stage.

Document AI is an advanced AI model to extract data from documents. It can read both text and images, like logos or signatures, and is perfect for processing documents like invoices or financial statements automatically.
More information about Document AI can be found in the official [Snowflake's Introduction to Document AI](https://docs.snowflake.com/en/user-guide/snowflake-cortex/document-ai/overview).

### Prerequisite for DocumentAI node type

* Set up the required objects(database,schema) and privileges to create table,task,stream
* Prepare and publish a [DocumentAI model](https://docs.snowflake.com/en/user-guide/snowflake-cortex/document-ai/tutorials/create-processing-pipelines#prepare-a-document-ai-model-build) in Snowflake using DocumentAI interface

### Usage of DocumentAI node type
* Set up the required objects and privileges
* Prepare and publish a Document AI model build in Snowflake using DocumentAI interface
* Provide the information of the Document AI model build under config section of the node added for Document AI node type.Also provide task related information as well
* The node creates a pipeline to process documents
* The data is available in the target table only after uploading new documents to the internal stage specified in config.
* If the node is created with 'Development mode-ON',no task is created and data can instantly loaded into target from documents using run option once the files are uploaded.
* If the node is created with 'Development mode-OFF',task is created to process the uploaded files 

### Document AI Node Configuration

The Document AI node has the following configuration groups:

* [Node Properties](#stream-and-insert-or-merge-node-properties)
* [General Options](#Document-AI-general-options)
* [Stream Options](#Document-AI-stream-options)
* [Source Data](#Document-AI-Source-Data)
* [Document AI Model build](#Document-AI-Model-build)
* [Scheduling Options](#Document-AI-scheduling-options)


#### Document AI Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the stream,table,task will be created |
| **Node Type** | Name of template used to create node objects |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### Document AI General Options

| **Option** | **Description** |
|------------|----------------|
| **Development Mode** | True / False toggle that determines whether a task will be created or if SQL executes as DML<br/>**True** - Table created and SQL executes as Run action<br/>**False** - SQL wrapped in task with specified Scheduling Options |
| **CREATE AS** | Choose target object type:<br/>- Table : Permanent table with data retention and fail-safe<br/>- Transient Table : Temporary table without data retention |
| **Truncate Before** | True / False toggle determines whether a table will be overwritten each time a task executes<br/>**True** - Uses INSERT OVERWRITE<br/>**False** - Uses INSERT to append data |


#### Document AI Stream Options

| **Option** | **Description** |
|------------|----------------|
| **Source Object** | **Directory Table**:<br/>- A directory table is an object that sits on top of a stage, similar to an external table, and stores metadata about the files in the stage. It doesn’t have its own privileges and is used to reference file-level data. Both external (cloud storage) and internal (Snowflake) stages support directory tables. You can add a directory table to a stage when creating it with CREATE STAGE or modify it later using ALTER STAGE. |
| **Redeployment Behavior** | options for Redeployment : <br/>- Create or Replace<br/>- Create if Not Exists<br/>- Create at Existing Stream |

#### Document AI Source Data

| **Option** | **Description** |
|------------|----------------|
| **Colaesce Storage Location of stage** | The Storage location Name in Coalesce where the stage is located |
| **Stage Name** | The Stage name created in Snowflake |

#### Document AI Model build

| **Option** | **Description** |
|------------|----------------|
| **Coalesce Storage Location of DocumentAI model** | The Storage location Name in Coalesce where the extraction query of DOcumentAI model is located |
| **DocumentAI model Name** | The identifier or name of the a [model build](https://docs.snowflake.com/en/user-guide/snowflake-cortex/document-ai/prepare-model-build) for extracting information <br/>(Located in AI & ML- Document AI in Snowflake) |
| **DocumentAI model build Version** | The specific [version of the DocumentAI model](https://docs.snowflake.com/en/user-guide/snowflake-cortex/document-ai/overview#document-ai-model-version-history), helping to track updates or changes<br/>(Build Version in AI & ML- Document AI in Snowflake)|

#### Document AI Scheduling Options

| **Option** | **Description** |
|------------|----------------|
| **Scheduling Mode** | Choose compute type:<br/>- **Warehouse Task**: User managed warehouse executes tasks<br/>- **Serverless Task**: Uses serverless compute |
| **When Source Stream has Data Flag** | True/False toggle to check for stream data<br/>**True** - Only run task if source stream has capture change data<br/>**False** -  Run task on schedule regardless of whether the source stream has data. If the source is not a stream should set this to false. |
| **Select Warehouse on which to run task** | Visible if Scheduling Mode is set to Warehouse Task. Enter the name of the warehouse you want the task to run on without quotes.|
| **Select initial serverless Warehouse size** | Visible when Scheduling Mode is set to Serverless Task.<br/> Select the initial compute size on which to run the task. Snowflake will adjust size from there based on target schedule and task run times. |
| **Task Schedule** | Choose schedule type:<br/>- **Minutes** - Specify interval in minutes.<br/>- **Cron** - . Specifies a cron expression and time zone for periodically running the task. Supports a subset of standard cron utility syntax.|
| **Enter task schedule using minutes**| Visible when Task Schedule is set to Minutes.<br/> Enter a whole number from 1 to 11520 which represents the number of minutes between task runs |
| **Enter task schedule using CRON**| For more reference visit [Cron expressions](https://docs.coalesce.io/docs/reference/cron-reference/) |

### DocumentAI - System Columns

The set of columns which has source data and file metadata information.

| **System Column**        | **Description** |
|--------------------------|------------------|
|**FILENAME**               | Name of the staged documents extracted|
| **FILE_URL**              | The location url of the document|
| **FILE_LAST_MODIFIED**    | The last modified timestamp of the staged documents|
| **SIZE**                  | The size of the document extracted|
| **EXTRACTED_DATA**        | The data extracted from documents|
| **DATA_EXTRACT_TIMESTAMP**| The load timestamp of document extraction|

### Document AI Deployment

#### Document AI Deployment Parameters

The DocumentAI node includes an environment parameter that allows you to specify a different warehouse used to run a task in different environments.

The parameter name is `targetTaskWarehouse` with default value `DEV ENVIRONMENT`.

```json
{
    "targetTaskWarehouse": "DEV ENVIRONMENT"
}
```

When set to any value other than DEV ENVIRONMENT` the node will attempt to create the task using a Snowflake warehouse with the specified value.

For example, with the below setting for the parameter in a QA environment, the task will execute using a warehouse named `SNOWFLAKE_DOCUMENT_AI_WH`.

```json
{
    "targetTaskWarehouse": "SNOWFLAKE_DOCUMENT_AI_WH"
}
```

#### Document AI Initial Deployment


| **Stage** | **Description** |
|-----------|----------------|
| **Create Stream** | Creates Stream in target environment |
| **Create Work Table/Transient Table** | Creates table loaded by task |
| **Create Task** | Creates scheduled task |
| **Resume Task** | Enables task execution |

If a task is part of a DAG of tasks, the DAG needs to include a node type called `Task DAG Resume Root`. This node will resume the root node once all the dependent tasks have been created as part of a deployment.
The task node has no ALTER capabilities. All task-enabled nodes are CREATE OR REPLACE only, though this is subject to change

#### Document AI Redeployment

Stream redeployment behavior:

| **Redeployment Behavior** | **Stage Executed** |
|--------------------------|-------------------|
| **Create Stream if not exists** | Re-Create Stream at existing offset |
| **Create or Replace** | Create Stream |
| **Create at existing stream** | Re-Create Stream at existing offset |

Table changes execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Rename Table/Alter Column/Delete Column/Add Column/Edit description** | Alters table as needed |

If the materialization type is changed from one type to another(transient table/table) the following stages execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Drop Table/Transient Table** | Drop the target table|
| **Create Work/Transient table**| Create the target table|

Task changes:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Task** | Creates scheduled task |
| **Resume Task**| Resumes the task|

### Redeployment with no changes 

If the nodes are redeployed with no changes compared to previous deployment,then no stages are executed

### Document AI Undeployment

When node is deleted, the following stages execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Drop Stream** | Removes the stream |
| **Drop Table** | Drop the table |
| **Drop Current Task** | Drop the task |

## Cortex Search Service

The Cortex Search UDN enables low-latency, high-quality “fuzzy” search over your Snowflake data. Cortex Search powers a broad array of search experiences for Snowflake users including Retrieval Augmented Generation (RAG) applications leveraging Large Language Models (LLMs).

[Cortex Search](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-search/cortex-search-overview) gets you up and running with a hybrid (vector and keyword) search engine on your text data in minutes, without having to worry about embedding, infrastructure maintenance, search quality parameter tuning, or ongoing index refreshes.

### Usage of Cortex Search Service node type
* Set up the required objects and privileges
* To create Cortex Search Service,keep 'Create Cortex Search Service' toggle ON.Provide schedule and advanced options.Hit create button and CSS is created
* To preview/query the Cortex Search Service,keep 'Preview Cortex Search Service' toggle ON.Provide the necessary configs.Hit run button and a target view with the search results
* You can create a Cortex Search Service once on a source data and use the same to create multiple target views
* 
![cssmultiple](https://github.com/user-attachments/assets/51b99f16-059a-440d-8a58-c6aed82ad2a1)

### Cortex Search Service Node Configuration

The Cortex Search service node has the following configuration groups:

* [Node Properties](#stream-and-insert-or-merge-node-properties)
* [General Options](#cortex-search-service-general-options)
* [Cortex search schedule Options](#cortex-search-schedule-options)
* [Cortex search Advanced Options](#cortex-search-advanced-options)
* [Preview search Options](#preview-search-options)

![cssnodeconfig](https://github.com/user-attachments/assets/123b80e6-4697-4824-b1ec-34d6925f5f33)

#### Cortex Search service Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the stream,table,task will be created |
| **Node Type** | Name of template used to create node objects |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### Cortex Search service General Options

| **Option** | **Description** |
|------------|----------------|
| **Create Cortex Service** | True / False toggle that determines whether to create a cortex search service <br/>**True** -  SQL executes as Create action to create cortex search service <br/>**False** - No cortex search service created |
| **Preview/Query the Cortex Service** | True / False toggle that determines whether to query the cortex search service created <br/>**True** -  SQL executes as run action to create view with the results of querying the cortex search service <br/>**False** - No view is created with the query results of cortex search service |

![cssgeneral](https://github.com/user-attachments/assets/1c051a9d-4c7f-4ef9-8415-d31ae035c9c8)

#### Cortex search schedule Options

| **Option** | **Description** |
|------------|----------------|
| **Warehouse** | (Required) Name of warehouse used to refresh the Dynamic Table |
| **Lag Specification** |(Required) Specifies the maximum amount of time that the Cortex Search service content should lag behind updates to the base tables specified in the source query.Set refresh schedule with:<br/>- **Time Value**: Frequency of the refresh<br/>- **Time Period**: Seconds/Minutes/Hours/Days |
| **Initialize** | (Required) Initial refresh behavior:<br/>- **ON_CREATE**: Refresh synchronously at creation<br/>- **ON_SCHEDULE**: Refresh at next scheduled time |

![cssschedule](https://github.com/user-attachments/assets/ba0e3bc5-633a-4db4-9b08-b3ffc4cc2e4a)

#### Cortex search Advanced Options

| **Option** | **Description** |
|------------|----------------|
| **Embedding model** | (Required)Parameter that specifies the embedding model to use in the Cortex Search Service. |
| **Search column** |(Required) Specifies the text column in the base table that you wish to search on. This column must be a text value. |
| **Attribute Columns** | (Required)Specifies comma-separated list of columns in the base table that you wish to filter on when issuing queries to the service.|

![cssadvanced](https://github.com/user-attachments/assets/571dce34-2aee-49ac-95b6-382f51acf705)

#### Preview search options

| **Option** | **Description** |
|------------|----------------|
| **Use existing Cortex Search Service** |It allows to use an existing Cortes search service with same source information  |
| **All Columns in the preview** |It allows all columns part of search service to be part of target view|
| **Columns in Target View**|It allows to choose only specific columns to be part of target view.If the above toggle is true,this option is ddisabled|
| **Filter**|Cortex Search supports filtering on the ATTRIBUTES columns specified|
| **Multiple filter conditions**|Allows you to add multiple filter criteria|
| **Apply numeric boost to Search Service**|Numeric boosts are applied as weighted averages to the returned fields, while decays leverage a log-smoothed function to demote less recent values.|
| **Apply time decay to Search Service**|Date or time metadata that boosts more recent results. The influence of recency signals decays over time.|
| **Query**|It searches query against the created cortex seacrh service|

![previewsearch](https://github.com/user-attachments/assets/454bcd06-cffe-4341-9774-93390227b9a3)

#### Cortex Search Service Deployment Parameters

The Cortex Search Service node includes an environment parameter that allows you to specify a different warehouse used to run a task in different environments.

The parameter name is `searchServiceWarehouse` with default value `DEV ENVIRONMENT`.

```json
{
    "searchServiceWarehouse": "DEV ENVIRONMENT"
}
```

When set to any value other than DEV ENVIRONMENT` the node will attempt to create the task using a Snowflake warehouse with the specified value.

For example, with the below setting for the parameter in a QA environment, the task will execute using a warehouse named `COMPUTE_WH`.

```json
{
    "searchServiceWarehouse": "COMPUTE_WH"
}
```
#### Cortex Search Service Redeployment

After initial deployment, subsequent deployments may alter or recreate the Cortex Search Service.

#### Altering the Cortex Search Service

The following config changes trigger ALTER statements:

1. Warehouse name
2. Description of Cortex Search Service
3. Lag specification

These execute the two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Alter Cortex Search Service** | Alters Cortex Search service |
| **Refresh Cortex Search Service** | Refreshes Cortex Search Service |

Other column or table level changes like data type change, column name change, column addition/deletion or config level changes result in a `CREATE` statement.

### Redeployment with no changes 

If the nodes are redeployed with no changes compared to previous deployment,then no stages are executed

### Cortex Search Service Undeployment

A Cortex Search Service will be dropped if all of these are true:

* The  Node is deleted from a Workspace.
* The Workspace is committed to Git.
* The Workspace committed to Git is deployed to a higher level environment.

| **Stage** | **Description** |
|-----------|----------------|
| **Drop Create Search Service** | Removes table from target environment |

---

## Code

### ML Forecast Code

* [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Forecast-262/definition.yml)
* [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Forecast-262/create.sql.j2)
* [Run Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Forecast-262/run.sql.j2)

### ML Anomaly Detection Code

* [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/AnomalyDetection-261/definition.yml)
* [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/AnomalyDetection-261/create.sql.j2)
* [Run Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/AnomalyDetection-261/run.sql.j2)

### LLM Cortex Functions Code

* [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/CortexFunctions-271/definition.yml)
* [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/CortexFunctions-271/create.sql.j2)
* [Run Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/CortexFunctions-271/run.sql.j2)

### TopInsights Code

* [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/TopInsights-279/definition.yml)
* [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/TopInsights-279/create.sql.j2)
  
### Classification Code

* [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Classification-337/definition.yml)
* [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Classification-337/create.sql.j2)
* [Run Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Classification-337/run.sql.j2)

### Document AI Code

* [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/DocumentAI-429/definition.yml)
* [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/DocumentAI-429/create.sql.j2)
* [Run Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/DocumentAI-429/run.sql.j2)

### Cortex Search Service

*[Node_definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Cortexsearchservice-499/definition.yml)
*[Create_Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Cortexsearchservice-499/create.sql.j2)
*[Run_Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Cortexsearchservice-499/run.sql.j2)

[Macros](https://github.com/coalesceio/Cortex-Node-types/blob/main/macros/macro-1.yml)
