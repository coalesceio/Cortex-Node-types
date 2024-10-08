# Cortex Package

 * [ML Forecast](#ml-forecast)
 * [ML Anomaly Detection](#ml-anomaly-detection)
 * [Cortex functions](#llm-cortex-functions)
 * [Top Insights](#topinsights)
 * [Classification](#classification)
 * [Code](#code)
   
<h2 id="ml-forecast"> ML Forecast </h2>

The Coalesce ML Forecast UDN is a versatile node that allows you to create a forecast table and insert forecasts of time series data using
the Snowflake built-in class [FORECAST](https://docs.snowflake.com/sql-reference/classes/forecast).

[Snowflake Cortex](https://docs.snowflake.com/en/user-guide/ml-powered-forecasting) is Snowflake’s intelligent, fully-managed service that enables organizations to quickly analyze data and build AI applications, all within Snowflake. This service makes Machine Learning (ML) functionality accessible to data engineers to enrich data pipelines while still using SQL. Forecasting employs a machine learning algorithm to predict future data by using historical time series data.

### ML Forecast Node Configuration
The ML Forecast has two configuration groups:

* [Node Properties](#ml-forecast-node-properties)
* [Forecast Model Input](#ml-forecast-model-input)


<h4 id="ml-forecast-node-properties"> ML Forecast Node Properties</h4>

* **Storage Location**: Storage Location where the Forecast table will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="ml-forecast-model-input"> ML Forecast Model Input </h4>

* **Model Instance Name (required):** Name of the model that needs to be created.
* **Create Model :** True or False Toggle to determine whether a model needs to be created or if an existing one can be referred.
  * **True** -  Allows you to forcefully create Forecast model.
  * **False** - Allows you to refer to an existing Forecast model.
* **Multi-Series Forecast:** True or False Toggle to determine if a single-series forecast or a multi-series forecast is to be created.
  * **True** -  Allows you to specify the series column,timestamp column and target column to create a multi-series forecast model.
	  * **Series Column (required for multi-series):** For multiple time series models, the name of the column defining the multiple time series in input data.
  * **False** - Allows you to specify the timestamp column and target column to create a single-series forecast model.
* **Timestamp Column (required):** Name of the column containing the timestamps in input data.
* **Target Column (required):** Name of the column containing the target (dependent value) in input data.
* **Config object:** An OBJECT containing key-value pairs used to configure the forecast job.
* **Series value:** Required for multi-series forecasts.Can be a single value or a VARIANT. 
* **Exogenous Variables:**
  * **True** -  Allows you to add future-valued exogenous data using multi-source toggle.
  * **False** - Allows you to create forecast model based on days to forecast without considering any exogenous variables.
* **Multi Source:** Future-valued Exogenous data can be added by enabling the multi source toggle.  
* **Days to Forecast:** Required for forecasts without exogenous variables.The number of steps ahead to forecast.

### ML Forecast Preprocessing of Data

When the forecast model returns an error, the error message returned by Snowflake is captured and surfaced directly in the Coalesce application for troubleshooting.

Here are common scenarios you may encounter:

* NULLS in the source data. The model will cope with some, but not too many NULLS.
* Missing time periods. If the model is unable to determine a consistent frequency in the time series it will cause an error.
* Missing exogenous variables. If the model was trained with exogenous variables.
* Exogenous variables need to be provided into the future to predict future values.
  
A data preparation step in Coalesce can be used prior to the ML Forecast node to address these issues.

### ML Forecast Deployment

#### ML Forecast Initial Deployment

When deployed for the first time into an environment the ML Forecast node will execute the below stage:

  **Create Forecast Table**

This stage will execute a `CREATE OR REPLACE` statement and create a Forecast Table in the target environment.

#### ML Forecast Redeployment

After the ML Forecast node has been deployed for the first time into a target environment, subsequent deployments may result in altering the forecast table.

#### Altering the Forecast Tables

The following column or table changes that is made in isolation or all-together will result in an ALTER statement to modify the Forecast table in the target environment.

* Change in table name
* Dropping existing column
* Alter column data type
* Adding a new column 

The following stages are executed:

* **Clone Table :** Creates an internal table
* **Rename Table /Alter Column/Delete Column/Add Column/Edit table description:** Alter table statement is executed to perform the alter operation accordingly.
* **Swap cloned Table:** Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost.
* **Delete Table:** Drops the internal table

### ML Forecast Undeployment

If a ML Forecast table is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Forecast Table in the target environment will be dropped.

This is executed in two stages:

* **Delete Table:** Coalesce Internal table is dropped
* **Delete Table:** Target table in Snowflake is dropped


<h2 id="ml-anomaly-detection">ML Anomaly Detection</h2>

The Coalesce ML Anomaly Detection UDN is a versatile node that allows you to create a Anomaly table and insert anomalies of time series data using
the Snowflake built-in class [ANOMALY DETECTION](https://docs.snowflake.com/en/sql-reference/classes/anomaly_detection).

[Snowflake Cortex](https://docs.snowflake.com/en/user-guide/ml-powered-anomaly-detection) is Snowflake’s intelligent, fully-managed service that enables organizations to quickly analyze data and build AI applications, all within Snowflake. This service makes Machine Learning (ML) functionality accessible to data engineers to enrich data pipelines while still using SQL. Anomalies in data are detected by analyzing the dataset using a machine learning algorithm.

### ML Anomaly Detection Node Configuration
The ML Anomaly  has two configuration groups:

* [Node Properties](#ml-anomaly-node-properties)
* [Anomaly Model Input](#ml-anomaly-model-input)

<h4 id="ml-anomaly-node-properties"> ML Anomaly Detection Node Properties </h4>
There are four configs within the **Node Properties** group.

* **Storage Location**: Storage Location where the Table will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="ml-anomaly-model-input"> ML Anomaly Model Input</h4>

* **Model Instance Name (required):** Name of the model that needs to be created.
* **Create Model :** True or False Toggle to determine whether a model needs to be created or if an existing one can be referred.
  * **True** -  Allows you to forcefully create Anomaly model.
  * **False** - Allows you to refer to an existing Anomaly model.
* **Multi-Series:** True or False Toggle to determine if a single-series or a multi-series  is to be created.
  * **True** -  Allows you to specify the series column,timestamp column and target column to create a multi-series Anomaly model.
	  * **Series Column (required for multi-series):** For multiple time series models, the name of the column defining the multiple time series in input data.
  * **False** - Allows you to specify the timestamp column and target column to create a single-series Anomaly model.
* **Timestamp Column (required):** Name of the column containing the timestamps in input data.
* **Target Column (required):** Name of the column containing the target (dependent value) in input data.
* **Config object:** An object that contains configuration settings for configuring the anomaly detection job.
* **Supervised Data :** Allows you to train an anomaly detection model using labeled data through the  multi-source toggle.
	* **Labeled Column:** Essential for supervised data, this column distinguishes between normal and anomalous instances.
* **Unsupervised Data :** Allows you to train an anomaly detection model using historical data through the  multi-source toggle.
* **Multi Source:** Data that needs to be analyzed can be added by enabling the multi source toggle.  
  
### ML Anomaly Detection Preprocessing of Data 

When the Anomaly model returns an error, the error message returned by Snowflake is captured and surfaced directly in Coalesce for troubleshooting.

Here are common scenarios you may encounter:

* Missing time periods. If the model is unable to determine a consistent frequency in the time series it will cause an error.
* Missing labeled column. If the model was trained with supervised data it's, necessary to pass a labeled column.
  
Often, a data preparation step in Coalesce can be used prior to the ML Anomaly Detection node to address these issues.

### ML Anomaly Detection  Deployment

#### ML Anomaly Detection  Initial Deployment

When deployed for the first time into an environment the ML Anomaly Detection node will execute the below stage:

  **Create Anomaly Detection Table**

This stage will execute a `CREATE OR REPLACE` statement and create a Anomaly Table in the target environment.

#### ML Anomaly Detection Redeployment

After the ML Anomaly node has been deployed for the first time into a target environment, subsequent deployments may result in altering the Anomaly table.

#### Altering the Anomaly Tables

The following column or table changes that is made in isolation or all-together will result in an ALTER statement to modify the Anomaly table in the target environment.

* Change in table name
* Dropping existing column
* Alter column data type
* Adding a new column 

The following stages are executed:

* **Clone Table :** Creates an internal table
* **Rename Table /Alter Column/Delete Column/Add Column/Edit table description:** Alter table statement is executed to perform the alter operation accordingly.
* **Swap cloned Table:** Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost.
* **Delete Table:** Drops the internal table.

### ML Anomaly Detection  Undeployment

If a ML Anomaly table is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Anomaly Table in the target environment will be dropped.

This is executed in two stages:

* **Delete Table:** Coalesce Internal table is dropped.
* **Delete Table:** Target table in Snowflake is dropped.

<h2 id="llm-cortex-functions"> LLM Cortex Functions</h2>

The Coalesce Cortex Function UDN is a versatile node that provides instant access to industry-leading large language models (LLMs) developed by researchers.
Additionally, it offers models that Snowflake has finely tuned for specific use cases.

[Snowflake Cortex - LLM Functions](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions) LLMs are fully hosted and managed by Snowflake, using them requires no setup. Your data stays within Snowflake, giving you the performance, scalability, and governance you expect.

### LLM Cortex Functions Node Configuration
The LLMs Cortex function has three configuration groups:

* [Node Properties](#llm-cortex-functions-node-properties)
* [Options](#llm-cortex-functions-options)
* [Cortex Package](#llm-cortex-functions-cortex-package)

<h4 id="llm-cortex-functions-node-properties">LLM Cortex Functions Node Properties  </h4>

* **Storage Location**: Storage Location where the Table will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="lm-cortex-functions-options">LLM Cortex Functions Options</h4>

* **Create As**: Provides option to choose materialization type as `table`.
* **Multi Source**: True / False toggle that is Coalesce implementation of SQL UNIONs.
    * True - Multiple sources can be combined in a single node. The sources are combined using the option specified in the Multi Source Strategy.
    * False - Single source node or multiple sources combined using a join.
* **Truncate Before**: True / False toggle that determines whether or not a table is overwritten each time a task executes.
    * True - INSERT OVERWRITE is used to overwrite existing data with new data loaded by task.
    * False - INSERT is used to append new data into target table.
* **Enable tests**:
	* **Pre-SQL**: Any SQL to be executed as a predecessor to data insert operation can be mentioned here.
	* **Post-SQL**: Any SQL to be executed post the data insert operation can be specified here.

<h4 id="llm-cortex-functions-cortex-package"> LLM Cortex Functions Cortex Package </h4>
  
* **SUMMARIZE :** Toggle to indicate if data from the mentioned column should be returned as a summary.
	* True - The system will prompt to add a column.
	* False- The function will remain inactive.
* **SENTIMENT :** Toggle to return sentiment as a score between -1 to 1 (with -1 being the most negative and 1 the most positive, with values around 0 neutral) for the given English-language input text.
	* True - The system will prompt to add a column.
	* False- The function will remain inactive.
* **TRANSLATE :** Toggle to translates text from the indicated or detected source language to a target language. 
	* True - The system will prompt to add a column.
	* False- The function will remain inactive.
* **EXTRACT ANSWER :** Toggle to indicate whether to extract an answer to a given question.
	* True - The system will prompt to add a column.
	* False- The function will remain inactive.   

 
### LLM Cortex Functions Key Points for Consideration

* Review the [required privileges](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions).
* Datatype of target column for using function "Extract Answer" should be an Array.
* If source data is required in the target column, duplication of column is necessary before applying a cortex function.
  
### LLM Cortex Functions Deployment

#### LLM Cortex Functions Initial Deployment

When deployed for the first time into an environment the llm node will execute the below :

  **Create Table**

This  will execute a `CREATE OR REPLACE` statement and create a  Table in the target environment.

#### LLM Cortex Functions Redeployment

After the LLM node has been deployed for the first time into a target environment, subsequent deployments may result in altering the table.

#### Altering the LLM Cortex Functions Tables

The following column or table changes if made in isolation or all-together will result in an ALTER statement to modify the table in the target environment. 

* Change in table name
* Dropping existing column
* Alter Column data type
* Adding a new column

The following stages are executed:

* **Clone Table :** Creates an internal table
* **Rename Table /Alter Column/Delete Column/Add Column/Edit table description:** Alter table statement is executed to perform the alter operation accordingly.
* **Swap cloned Table:** Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost.
* **Delete Table:** Drops the internal table.

### LLM Cortex Functions Undeployment

If a LLM Node is deleted from a Workspace, and that Workspace is committed to Git, subsequently deployed to a higher-level environment, then the table in the target environment will be dropped.

This is executed in two stages:

* **Delete Table:** Coalesce Internal table is dropped
* **Delete Table:** Target table in Snowflake is dropped

<h2 id="topinsights">Top Insights</h2>

The Coalesce Top Insights UDN is a versatile node that allows you to streamline and improve the process of root cause analysis around changes in observed metrics. Learn more about [Top Insights](https://docs.snowflake.com/en/user-guide/snowflake-cortex/ml-functions/contribution-explorer).

###  Top Insights Node Configuration
The Top Insights node has two configuration groups:

* [Node Properties](#ml-contribution-explore-node-properties)
* [Top Insights](#ml-contribution-explorer-config)

<h4 id="ml-contribution-explorer-node-properties">Top Insights Node Properties </h4>

* **Storage Location**: Storage Location where the Table will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="ml-contribution-explorer-config">Top Insights Configuration</h4>

* **Create As**:Provides option to create a ‘view’,
* **CATEGORICAL DIMENSIONS (required):**  This column denotes a categorical attributes essential for analysis.
* **CONTINUOUS DIMENSIONS (required ):** The Columns representing a continuous aspect that varies within a range.
* **Metric (required):** Column representing a target metric that is being investigated.
* **Label (required):**  Column that distinguishes between control and test data. TRUE represents test data, and FALSE represents control data.
* **Filter Insights:**  A textbox to customize the filtering of top insights according to the preferences.
* **Order By** : True/False toggle that determines whether to add “ORDER BY” to SQL Query along with the column and sort order.
    * True -Sort column and sort order drop down are visible and are required to form order by clause.
    * False-Sort column and sort order drop down are invisible.

### Top Insights  Deployment

#### Top Insights  Initial Deployment

When deployed for the first time into an environment the View node will execute the below stage:

  **Create or replace View**

This stage will execute a `CREATE OR REPLACE` statement and create a View in the target environment.

#### Top Insights Redeployment

The subsequent deployment of a View node with changes in the view definition, altering options, or renaming the view results in the deletion of the existing view and the recreation of the view.

The following stages are executed:

* Delete View
* Create View

### Top Insights Undeployment

If a View Node is removed from a Workspace, and the changes are committed to Git and deployed to a higher-level environment, the corresponding View in the target environment will be dropped.

This is executed in the below stage:

* Delete View

<h2 id="classification"> Classification </h2>

The Coalesce Classification is a versatile node that allows you to create a classification table and classifaction model to classify data 
into different classes using patterns detected in training data using in-built snowflake ML function[CLASSIFICATION](https://docs.snowflake.com/en/sql-reference/classes/classification).

Classification uses machine learning algorithms to sort data into different classes using patterns detected in training data. Binary classification (two classes) and multi-class classification (more than two classes) are supported. Common use cases of classification include customer churn prediction, credit card fraud detection, and spam detection.

### Classification Node Configuration

The Classification node has two configuration groups:

* [Node Properties](#classification-node-properties)
* [Classification Model Input](#classification-model-input)

<h4 id="classification-node-properties"> Classification Node Properties </h4>
There are four configs within the **Node Properties** group.

* **Storage Location**: Storage Location where the Table will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="classification-model-input"> Classification Model Input</h4>

* **Model Instance Name (required):** Name of the model that needs to be created.
* **Create Model :** True or False Toggle to determine whether a model needs to be created or if an existing one can be referred.
  * **True** -  Allows you to forcefully create Classification model.
  * **False** - Allows you to refer to an existing Classification model.
* **Target Column (required):** Name of the column containing the target (dependent value) in input data.
* **Multi Source:** Data that needs to be analyzed can be added by enabling the multi source toggle.
  
### Classification  Deployment

#### Classification  Initial Deployment

When deployed for the first time into an environment the Classification node will execute the below stage:

  **Create Classification Table**

This stage will execute a `CREATE OR REPLACE` statement and create a Classification Table in the target environment.

#### Classification Redeployment

After the Classification node has been deployed for the first time into a target environment, subsequent deployments may result in altering the Classification table.

#### Altering the Classification Tables

The following column or table changes that is made in isolation or all-together will result in an ALTER statement to modify the Classification table in the target environment.

* Change in table name
* Dropping existing column
* Alter column data type
* Adding a new column 

The following stages are executed:

* **Clone Table :** Creates an internal table
* **Rename Table /Alter Column/Delete Column/Add Column/Edit table description:** Alter table statement is executed to perform the alter operation accordingly.
* **Swap cloned Table:** Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost.
* **Delete Table:** Drops the internal table.

### Classification  Undeployment

If a Classification table is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Classification Table in the target environment will be dropped.

This is executed in two stages:

* **Delete Table:** Coalesce Internal table is dropped.
* **Delete Table:** Target table in Snowflake is dropped.
  
## Code

### ML Forecast

*  [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Forecast-262/definition.yml)
*  [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Forecast-262/create.sql.j2)
*  [Run Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Forecast-262/run.sql.j2)

### ML Anomaly Detection

*  [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/AnomalyDetection-261/definition.yml)
*  [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/AnomalyDetection-261/create.sql.j2)
*  [Run Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/AnomalyDetection-261/run.sql.j2)

### LLM Cortex Functions

*  [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/CortexFunctions-271/definition.yml)
*  [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/CortexFunctions-271/create.sql.j2)
*  [Run Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/CortexFunctions-271/run.sql.j2)

### TopInsights

*  [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/TopInsights-279/definition.yml)
*  [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/TopInsights-279/create.sql.j2)
  
### Classification

*  [Node definition](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Classification-337/definition.yml)
*  [Create Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Classification-337/create.sql.j2)
*  [Run Template](https://github.com/coalesceio/Cortex-Node-types/blob/main/nodeTypes/Classification-337/run.sql.j2)

[Macros](https://github.com/coalesceio/Cortex-Node-types/blob/main/macros/macro-1.yml)
