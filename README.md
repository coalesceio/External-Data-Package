# External Data Package

The Coalesce External Data Package includes:

* [Inferschema](#inferschema)
* [CopyInto](#CopyInto)
* [Snowpipe](#Snowpipe)
* [External Table](#external-tables)
* [CopyUnload](#CopyUnload)
* [API](#API)
* [JDBC LOAD](#jdbc-load)
* [Parse Excel](#parse-excel)
* [Parse Json](#parse-json)
* [Code](#code)

<h2 id="inferschema"> InferSchema </h2>

The Coalesce InferSchema UDN is a versatile node infers schema of the file in internal or external stage and dynamically creates the target table of the same name as inferschema node.

[InferSchema](https://docs.snowflake.com/en/sql-reference/functions/infer_schema) in Snowflake automatically detects the file metadata schema in a set of staged data files that contain semi-structured data and retrieves the column definitions.

### InferSchema Prerequisites

* A sample file in the internal or external stage.
* An existing fileformat to parse the file
* The file format is also expected to be in the same location as stage

### InferSchema Node Configuration

The InferSchema node type has two configuration groups:

* [InferSchema Node Properties](#inferschema-node-properties)
* [InferSchema Source Data](#inferschema-source-data)

Go to the node and select the **Config tab** to see the Node Properties, Dynamic Table Options and General Options.

<h4 id="inferschema-node-properties"> InferSchema Node Properties </h4>

* **Storage Location**: Storage Location where the WORK will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.
  
<h4 id="inferschema-source-data"> InferSchema Source Data </h4>

* **Stage Storage Location (Required)**: A storage location in Coalesce where the stage is located.
* **Stage Name (Required)***: Internal or External stage where the files containing data to be loaded are staged.
* **File Names (Required)**: Use commas to seperate multiple files. File whose metatdata is to be inferred.
* **File Format Name (Required)***: Name of the file format object that describes the data contained in the staged files.It is expected in the same location as Stage.
* **Redeployment Behavior**:
  * **CREATE OR REPLACE**: Dynamically creates target table based on the inferred schema from file in staging area.
  * **ALTER EXISTING TABLE**: Dynamically alters inferred table by comparing the inferred schema of the same file (with changes if any)and created table.
* **DROP EXISTING TABLE**: Drops the table inferred.

### InferSchema Usage

#### Option1-With deployment
* Add a InferSchema node, for example `INFER_JSON`.
* Deploy the node. On deployment a table of the same name as the InferSchema node is created with columns inferred on parsing the file. For example, `InferSchema node:INFER_JSON,Inferred table:INFER_JSON`.
* Add the inferred table to the browser using Add Sources tab.
* Now we can add a Copy-Into Snowpipe or External table node on top of the inferred table to load staged files.
* If the structure of the file is changed, you can redeploy the InferSchema node with "Alter existing table" redeployment behaviour. The already inferred table is altered.
* Next, you can re-sync the columns of the inferred table and redeploy the CopyInto Copy-Into Snowpipe or External table node added on top of it.
> 🚧 Inferred table and Copy-into nodes/External table cannot be deployed together
>
> Infer Schema node, inferred table and Copy-into nodes/External table cannot be deployed together. First deploy the Infer Schema node. Then add the inferred table in browser, add Copy-into node on top of the table and then deploy the same.

#### Option2-Without deployment
* Add a InferSchema node, for example `INFER_JSON` and hit create.'Infer and Create table' stage runs and creates a table with the same name as InferSchema node.
* Add the inferred table to the browser using Add Sources tab.
* Now we can add a Copy-Into,Snowpipe or External table node on top of the inferred table to load staged files.
* Once we get the required metadata columns in our downstream node(Copy-Into,Snowpipe or External table),bulk edit the source in the node to be blank.Also delete the from clause in the Join tab.These information are not required for Copy-Into,Snowpipe or External table to  load data from files.
* Delete the inferred table/source table for the downstream nodes mentioned step2.
* Delete the inferschema node ,we created in step1 as we have derived all the columns required for the downstream nodes and the data for the same are derived from files.
* Eventually Inferschema nodes are not required to be deployed.It is sufficient to deploy the Copy-into,Snowpipe or External table nodes which holds the data from staged files


### InferSchema Deployment

#### InferSchema Initial Deployment

When deployed for the first time into an environment the InferSchema node will execute the stage:

* Stage executed: **Infer and Create target table**

#### InferSchema Redeployment

**Redeployment Behavior: Create or Replace**

| Redeployment Behavior | Stage Executed |
|---|---|
| Create or Replace | Infer and Create target table

If any of the Source Data options like Stage storage location, Stage name or filename are modified.
Then you can redeploy the Infer Schema node with redeployment behaviour “Create or Replace”.

> 📘 Info
> 
> You can go back to the browser and Re-sync columns of Inferred table, re-execute Copy-Into and redeploy

**Redeployment Behavior: Alter Existing Table**

| Redeployment Behavior | Stage Executed |
|---|---|
|Alter existing table| Infer and Alter target table

If all Source Data options remain same and only there are changes in the existing file structure, you can redeploy the Infer Schema node with redeployment behaviour “Alter existing table”.

**Redeployment Behavior: Drop Existing Table**

| Redeployment Behavior | Stage Executed |
|---|---|
|Drop existing table |Drop inferred table

If you want to drop the inferred table you can redeploy the Infer Schema node with redeployment behaviour “Drop existing table”.
  
### InferSchema Undeployment

If the InferSchema node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then no action takes place.
  
<h2 id="CopyInto"> CopyInto </h2>

### CopyInto Node Configuration

The Copy-Into node type the following configurations available:

* [Node Properties](#copy-into-node-properties)
* [General Options](#copy-into-general-options)
* [Source Data](#copy-into-file-location)
* [File Format](#copy-into-file-format)
* [Copy Options](#copy-into-copy-options)

<h3 id="copy-into-node-properties">CopyInto - Node Properties</h3>

* **Storage Location**: Storage Location where the target table will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.
    
<h3 id="copy-into-general-options">  CopyInto - General Options </h3>

![CopyInto-general options](https://github.com/user-attachments/assets/fa520f20-f609-40ed-8df0-0cf64885ea93)

* **Create As**: Select from the options to create as Table or Transient Table
    * Transient Table
    * Table

<h3 id="copy-into-file-location">  CopyInto - Source Data </h3>

![CopyInto-File location](https://github.com/user-attachments/assets/3f95f8d0-3c23-4a46-8242-fb81989c523b)

* **Internal or External Stage**
  * **Coalesce Storage Location of Stage**: A storage location in Coalesce where the stage is located.
  * **Stage Name (Required)**: Internal or External stage where the files containing data to be loaded are staged
  * **File Name(s)(Optional - Ex:'a.csv','b.csv')**: Specifies a list of one or more files names (separated by commas) to be loaded
  * **File Pattern (Optional - Ex:'.*hea.*[.]csv')**: A regular expression pattern string, enclosed in single quotes, specifying the file names or paths to match.

![CopyInto-file location2](https://github.com/user-attachments/assets/0c5949c4-0286-4250-91cf-38325e2c312a)

* **External location**
  * **External URI**: Enter the URI of the External location
  * **Storage Integration**: Specifies the name of the storage integration used to delegate authentication responsibility for external cloud storage to a Snowflake identity and access management (IAM) entity

<h3 id="copy-into-file-format"> CopyInto - File Format </h3>

![copy-into-file-format](https://github.com/user-attachments/assets/2a737c5f-3bb3-471a-9365-bf3251677415)

* **File Format Definition - File Format Name**:
  * **File Format Name**: Specifies an existing named file format to use for loading data into the table.
  * **Coalesce Storage Location of File Format**:Location in Coalesce pointing to the database and schema,the file format resides.
  * **File Type**:
    * CSV
    * JSON
    * ORC
    * AVRO
    * PARQUET
    * XML
* **File Format Definition - File Format Values**: 
  * **File Format Values** -Provides file format options for the File Type chosen.
  * **File Type**: Each file type has different configurations available.
    * **CSV**
    * **Compression**: String (constant) that specifies the current compression algorithm for the data files to be loaded.
    * **Record delimiter**:Characters that separate records in an input file
    * **Field delimiter**:One or more singlebyte or multibyte characters that separate fields in an input file
    * **Field optionally enclosed by**:Character used to enclose strings
    * **Number of header lines to skip**:Number of lines at the start of the file to skip.
    * **Skip blank lines**:Boolean that specifies to skip any blank lines encountered in the data files.
    * **Trim Space**: Boolean that specifies whether to remove white space from fields.
    * **Replace invalid characters**: Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **Date format**:String that defines the format of date values in the data files to be loaded. 
    * **Time format**: String that defines the format of time values in the data files to be loaded
    * **Timestamp format**:String that defines the format of timestamp values in the data files to be loaded.
    * **JSON**
      * **Compression**: String (constant) that specifies the current compression algorithm for the data files to be loaded.
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
      * **Trim Space** - Boolean that specifies whether to remove white space from fields.
      * **Strip Outer Array**:Boolean that instructs the JSON parser to remove outer brackets [ ].
      * **Date format**:String that defines the format of date values in the data files to be loaded. 
      * **Time format**:String that defines the format of time values in the data files to be loaded
      * **Timestamp format**: String that defines the format of timestamp values in the data files to be loaded.
    * **ORC**
      * **Trim Space** - Specifies whether to remove white space from fields
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **AVRO**
      * **Trim Space** - Boolean that specifies whether to remove white space from fields.
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **PARQUET**
      * **Trim Space** - Boolean that specifies whether to remove white space from fields.
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **XML**
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.

<h3 id="copy-into-copy-options"> CopyInto - Copy Options </h3>

* **On Error Behavior**:String (constant) that specifies the error handling for the load operation.
  * CONTINUE
  * SKIP_FILE
  * SKIP_FILE_num
  * SKIP_FILE_num%
  * ABORT_STATEMENT
* **Specify the number of errors that can be skipped**: Required when On Error Behavior is either `SKIP_FILE_num` or `SKIP_FILE_num%`. Specify the number of errors that can be skipped.
* **Size Limit**: Number (> 0) that specifies the maximum size (in bytes) of data to be loaded for a given COPY statement.
* **Purge Behavior**: Boolean that specifies whether to remove the data files from the stage automatically after the data is loaded successfully.
* **Return Failed Only**: Boolean that specifies whether to return only files that have failed to load in the statement result.
* **Force**: Boolean that specifies to load all files, regardless of whether they’ve been loaded previously and have not changed since they were loaded. 
* **Load Uncertain Files**: Boolean that specifies to load files for which the load status is unknown. The COPY command skips these files by default.
* **Enforce Length**: Boolean that specifies whether to truncate text strings that exceed the target column length
* **Truncate Columns**: Boolean that specifies whether to truncate text strings that exceed the target column length

### CopyInto - System Columns

The set of columns which has source data and file metadata information.

* **SRC** - The data from the file is loaded into this variant column.
* **LOAD_TIMESTAMP** - Current timestamp when the file gets loaded.
* **FILENAME** - Name of the staged data file the current row belongs to. Includes the full path to the data file.
* **FILE_ROW_NUMBER** - Row number for each record in the staged data file.
* **FILE_LAST_MODIFIED** - Last modified timestamp of the staged data file the current row belongs to
* **SCAN_TIME** - Start timestamp of operation for each record in the staged data file. Returned as TIMESTAMP_LTZ.

### Key points to use CopyInto node
* CopyInto node can be created by just clicking on Create node from browser if we want the data from the file to be loaded into single variant column in target table.
* CopyInto node can be added on top of an inferred table(table created by running the inferschema node) if you want to load data into specific columns as defined in the files.Refer to Inferschema to know more on how to use the node and add Copy-Into on top of it.

### CopyInto Deployment

#### CopyInto Deployment Parameters

The CopyInto-Snowpipe includes an environment parameter that allows you to specify if you want to perform a full load or a reload based on the load type when you are performing a Copy-Into operation.

The parameter name is `loadType` and the default value is ``.

```json
{
    "loadType": ""
}
```

When the parameter value is set to `Reload`, the data is reloaded into the table regardless of whether they’ve been loaded previously and have not changed since they were loaded.

#### CopyInto Initial Deployment
When deployed for the first time into an environment the Copy-into node of materialization type table will execute the below stage:

| Deployment Behavior  | Load Type | Stages Executed |
|--|--|--|
| Initial Deployment | ``|Create Table/Transient Table
| Initial Deployment | Reload| Truncate Target table </br> Create Table/Transient Table

### CopyInto Redeployment

#### Altering the CopyInto Tables

There are few column or table changes like Change in table name,Dropping existing column, Alter Column data type,Adding a new column if made in isolation or all-together will result in an ALTER statement to modify the Work Table in the target environment.

The following stages are executed:

* **Rename Table| Alter Column | Delete Column | Add Column | Edit table description**: Alter table statement is executed to perform the alter operation.

#### Copy-Into change in materialization type

When the materialization type of Copy-Into node is changed from table to transient table or viceversa,the below stages are executed:

* **Drop table/transient table**
* **Create transient table/table**

### CopyInto Undeployment

If the CopyInto-Snowpipe node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

* **Drop table/transient table**: Target table in Snowflake is dropped

<h2 id="Snowpipe">Snowpipe </h2>

The Coalesce Snowpipe node is a node that performs two operations. It can be used to load historical data using     
 [CopyInto](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table). Also the Snowpipe node can be used to create a pipe to auto ingest files from AWS, GCP, or Azure.

[Snowpipe](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro) enables loading data from files as soon as they’re available in a stage. 

This means you can load data from files in micro-batches, making it available to users within minutes, rather than manually executing COPY statements on a schedule to load larger batches.

### Snowpipe Node Configuration

The Snowpipe node type the following configurations available:

* [Node Properties](#snowpipe-node-properties)
* [General Options](#snowpipe-general-options)
* [Snowpipe Options](#snowpipe-snowpipe-options)
* [File Location](#snowpipe-file-location)
* [File Format](#snowpipe-file-format)
* [Copy Options](#snowpipe-copy-options)

![Snowpipe-config](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/14cfeadf-2996-4665-80a1-ecd528691326)

<h3 id="snowpipe-node-properties">Snowpipe Node Properties</h3>

* **Storage Location**: Storage Location where the target table will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.
  * 
<h3 id="snowpipe-general-options"> General Options </h3>

![CopyInto-general options](https://github.com/user-attachments/assets/fa520f20-f609-40ed-8df0-0cf64885ea93)

* **Create As**: Dropdown that helps us to create a table or Transiet table to load data from external stage.  
  * Table 
  * Transient Table 

<h3 id="snowpipe-snowpipe-options"> Snowpipe Options </h3>

![snowpipe-snowpipe-options](https://github.com/user-attachments/assets/7e341bb3-4a36-4d4a-b451-5cde6805dad6)

* **Enable Snowpipe**: Drop down that helps us to create a pipe to auto ingest files from external stage or validate the Copy-Into statement.
  * Enable Snowpipe - Load data auto ingesting files from AWS, Azure, or GCP. Choose your **Cloud Provider.**
    * AWS - AWS SNS Topic. Specifies the Amazon Resource Name (ARN) for the SNS topic for your S3 bucket.
    * Azure - Integration. Specifies the existing notification integration used to access the storage queue.
    * GCP -  Integration. Specifies the existing notification integration used to access the storage queue.
  * Test Copy Statement - To validate the Copy-into statement before we use it to create PIPE
* **Load historical data**:Loads the historic data into the target table by executing a COPY_INTO statement.

<h3 id="snowpipe-file-location"> Snowpipe File Location </h3>

![snowpipe-file-location](https://github.com/user-attachments/assets/70189562-bb34-49ba-9735-db63f34b271c)

* **Coalesce Stage Storage Location of stage(Required)**: A storage location in Coalesce where the stage is located.
* **Stage Name (Required)**: Internal or External stage where the files containing data to be loaded are staged.
* **File Names**: Enabled when 'Enable Snowpipe' under Snowpipe Options is toggled off. Specifies a list of one or more files names (separated by commas) to be loaded. For example, `'a.csv','b.csv'`.
* **File Pattern**:A regular expression pattern string, enclosed in single quotes, specifying the file names or paths to match. For example, `*hea.*[.]csv'`.
  
<h3 id="snowpipe-file-format"> Snowpipe File Format </h3>

![Snowpipe-file-format](https://github.com/user-attachments/assets/ae640901-16ce-4ed2-8e6c-efae278d3942)

* **File Format Definition - File Format Name**:
  * **File Format Name**: Specifies an existing named file format to use for loading data into the table.
  * **Coalesce Storage Location of File Format**: File format location in snowflake that is mapped as storage location in Coalesce
  * **File Type**:
    * CSV
    * JSON
    * ORC
    * AVRO
    * PARQUET
    * XML
* **File Format Definition - File Format Values**: 
  * **File Format Values** -Provides file format options for the File Type chosen.
  * **File Type**: Each file type has different configurations available.
    * **CSV**
    * **Compression**: String (constant) that specifies the current compression algorithm for the data files to be loaded.
    * **Record delimiter**:Characters that separate records in an input file
    * **Field delimiter**:One or more singlebyte or multibyte characters that separate fields in an input file
    * **Field optionally enclosed by**:Character used to enclose strings
    * **Number of header lines to skip**:Number of lines at the start of the file to skip.
    * **Skip blank lines**:Boolean that specifies to skip any blank lines encountered in the data files.
    * **Trim Space**: Boolean that specifies whether to remove white space from fields.
    * **Replace invalid characters**: Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **Date format**:String that defines the format of date values in the data files to be loaded. 
    * **Time format**: String that defines the format of time values in the data files to be loaded
    * **Timestamp format**:String that defines the format of timestamp values in the data files to be loaded.
    * **JSON**
      * **Compression**: String (constant) that specifies the current compression algorithm for the data files to be loaded.
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
      * **Trim Space** - Boolean that specifies whether to remove white space from fields.
      * **Strip Outer Array**:Boolean that instructs the JSON parser to remove outer brackets [ ].
      * **Date format**:String that defines the format of date values in the data files to be loaded. 
      * **Time format**:String that defines the format of time values in the data files to be loaded
      * **Timestamp format**: String that defines the format of timestamp values in the data files to be loaded.
    * **ORC**
      * **Trim Space** - Specifies whether to remove white space from fields
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **AVRO**
      * **Trim Space** - Boolean that specifies whether to remove white space from fields.
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **PARQUET**
      * **Trim Space** - Boolean that specifies whether to remove white space from fields.
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **XML**
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.

<h3 id="snowpipe-copy-options"> Snowpipe Copy Options </h3>

If you toggle Enable Snowpipe under Snowpipe Options to *ON*, these configuration options are available.

* **On Error Behavior**: String (constant) that specifies the error handling for the load operation.
  * CONTINUE
  * SKIP_FILE
  * SKIP_FILE_num
    * Specify the number of errors that can be skipped.
  * SKIP_FILE_num%
    * Specify the number of errors that can be skipped.
* **Enforce Length**: Boolean that specifies whether to truncate text strings that exceed the target column length.
* **Truncate Columns**: Boolean that specifies whether to truncate text strings that exceed the target column length.

If you toggle Enable Snowpipe under Snowpipe Options to *OFF*, these configuration options are available.

* **On Error Behavior**:String (constant) that specifies the error handling for the load operation.
  * CONTINUE
  * SKIP_FILE
  * SKIP_FILE_num
  * SKIP_FILE_num%
  * ABORT_STATEMENT
* **Specify the number of errors that can be skipped**: Required when On Error Behavior is either `SKIP_FILE_num` or `SKIP_FILE_num%`. Specify the number of errors that can be skipped.
* **Size Limit**: Number (> 0) that specifies the maximum size (in bytes) of data to be loaded for a given COPY statement.
* **Purge Behavior**: Boolean that specifies whether to remove the data files from the stage automatically after the data is loaded successfully.
* **Return Failed Only**: Boolean that specifies whether to return only files that have failed to load in the statement result.
* **Force**: Boolean that specifies to load all files, regardless of whether they’ve been loaded previously and have not changed since they were loaded. 
* **Load Uncertain Files**: Boolean that specifies to load files for which the load status is unknown. The COPY command skips these files by default.
* **Enforce Length**: Boolean that specifies whether to truncate text strings that exceed the target column length
* **Truncate Columns**: Boolean that specifies whether to truncate text strings that exceed the target column length

### Snowpipe System Columns

The set of columns which has source data and file metadata information.

* **SRC** - The data from the file is loaded into this variant column.
* **LOAD_TIMESTAMP** - Current timestamp when the file gets loaded.
* **FILENAME** - Name of the staged data file the current row belongs to. Includes the full path to the data file.
* **FILE_ROW_NUMBER** - Row number for each record in the staged data file.
* **FILE_LAST_MODIFIED** - Last modified timestamp of the staged data file the current row belongs to
* **SCAN_TIME** - Start timestamp of operation for each record in the staged data file. Returned as TIMESTAMP_LTZ.

### Key points to use CopyInto node
* Snowpipe node can be created by just clicking on Create node from browser if we want the data from the file to be loaded into single variant column in target table.
* Snowpipe node can be added on top of an inferred table(table created by running the inferschema node) if you want to load data into specific columns as defined in the files.Refer to Inferschema to know more on how to use the node and add Snowpipe node on top of it.


### Snowpipe Deployment

#### Snowpipe Deployment Parameters

The Snowpipe includes twp environment parameters.The parameter `loadType` allows you to specify if you want to perform a full load or a reload based on the load type 
when you are performing a Copy-Into operation.
The another parameter `targetIntegrationOrAwsSNSTopic` allows you to specify the AWS SNS Topic or Integration name to be used for auto ingesting files from Clous providers

The parameter name is `loadType` and the default value is ``.

```json
{
    "loadType": ""
}
```

When the parameter value is set to `Reload`, the data is reloaded into the table regardless of whether they’ve been loaded previously 
and have not changed since they were loaded.

The parameter name is targetIntegrationOrAwsSNSTopic and the default value is DEV ENVIRONMENT.

When set to DEV ENVIRONMENT the value entered in the Snowpipe Options config AWS SNS TOpic/Integration is used for ingesting files from Cloud providers.

```
{
    "targetIntegrationOrAwsSNSTopic": "DEV ENVIRONMENT"
}
```

When set to any value other than DEV ENVIRONMENT the node will use the specified value for AWS SNS TOpic/Integration.

For example, the Snowpipe node will use the specific value for auto ingestion.

```
{
    "targetIntegrationOrAwsSNSTopic": "arn:aws:sqs:us-east-1:832620633027:sf-snowpipe-AIDA4DXAOTPB7IF437EPD-lZe8ciYpPqGgC9KwWLmiIQ"
}
```

#### Snowpipe Initial Deployment

When deployed for the first time into an environment the Snowpipe node will execute the below stages depending on if Enable Snowpipe is enabled,'Load Historical Data' is enabled and the `loadType` parameter.

| Deployment Behavior  | Enable Snowpipe | Historical Load | Load Type | Stages Executed |
|--|--|---|--|--|
| Initial Deployment | Enable Snowpipe |true| ``|Create Table </br> Historical full load using CopyInto </br> Create Pipe </br> Alter Pipe 
| Initial Deployment | Enable Snowpipe |true| Reload|Create table </br> Truncate Target table </br> Historical full load using CopyInto </br> Create Pipe </br> Alter Pipe
| Initial Deployment | Enable Snowpipe |false| Reload or Empty|Create table </br> Truncate Target table </br> Create Pipe
| Initial Deployment | Test Copy Statement |false| Reload or Empty|Create table </br> Test Copy Statement-No pipe creation

### Snowpipe Redeployment

#### Altering the Snowpipe node

There are few column or table changes like Change in table name, Dropping existing column,  Alter Column data type, Adding a new column if made in isolation or all-together will result in an ALTER statement to modify the target Table in the target environment.Any table level changes or node config changes results in recreation of pipe

The following stages are executed:

* **Rename Table| Alter Column | Delete Column | Add Column| Edit table description |**: Alter table statement is executed to perform the alter operation.
* **Truncate target table**:The target table is truncated in case the Load Type parameter is set to 'Reload'
* **Historical full load using CopyInto**:Historical data are loaded if 'Load Historical' toggle is on.
* **Create Pipe**: Pipe is recreated if enable snowpipe option is true
* **Alter Pipe**:Pipe is refreshed if 'Load Historical' toggle is on.

### Snowpipe Undeployment

If the Snowpipe node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

This is executed in two stages:

* **Drop Table**: Target table is dropped
* **Drop Pipe**: Pipe is dropped

## External Tables

The Coalesce External Table nodes create a new external table in the current/specified schema or replaces an existing external table.

An [external table](https://docs.snowflake.com/en/sql-reference/sql/create-external-table#examples) reads data from a set of one or more files in a specified external stage which can point to AWS,GCP or Azure cloud providers.

### External Tables Node Configuration

The External table node type has four configuration groups:

* [Node Properties](#external-tables-node-properties)
* [File Location](#external-tables-file-location)
* [File Format](#external-tables-ile-format)
* [Additional Options](#external-tables-additional-options)

<h4 id="external-tables-node-properties"> External Tables Node Properties</h4>

* **Storage Location**: Storage Location where the WORK will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="external-tables-file-location"> External Tables Node File Location </h4>

![Filelocation](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/48b7a0dd-301c-4077-857c-e37fd234bbf8)

* **Stage Storage Location (Required)**: A storage location in Coalesce where the stage is located.
* **Stage Name (Required)**: Internal or External stage where the files containing data to be loaded are staged.
* **File Pattern**:A regular expression pattern string, enclosed in single quotes, specifying the file names or paths to match. For example, `*hea.*[.]csv'`.

<h4 id="external-tables-ile-format">External Tables  File Format</h4>

![fileformat](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/82945bb5-ba5d-46dd-b66d-3d6c10ff77d9)


* **File Format Definition - File Format Name**:
  * **File Format Name**: Specifies an existing named file format to use for loading data into the table.
  * **File Type**:
    * CSV
    * JSON
    * ORC
    * AVRO
    * PARQUET
* **File Format Definition - File Format Values**: 
  * **File Format Values** -Provides file format options for the File Type chosen.
  * **File Type**: Each file type has different configurations available.
    * **CSV**
    * **Compression**: String (constant) that specifies the current compression algorithm for the data files to be loaded.
    * **Record delimiter**: Characters that separate records in an input file
    * **Field delimiter**: One or more singlebyte or multibyte characters that separate fields in an input file
    * **Field optionally enclosed by**: Character used to enclose strings
    * **Number of header lines to skip**: Number of lines at the start of the file to skip.
    * **Skip blank lines**:Boolean that specifies to skip any blank lines encountered in the data files.
    * **Trim Space**: Boolean that specifies whether to remove white space from fields.
    * **Replace invalid characters**: Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **JSON**
      * **Compression**: String (constant) that specifies the current compression algorithm for the data files to be loaded.
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
      * **Strip Outer Array**:Boolean that instructs the JSON parser to remove outer brackets [ ].
      * **Date format**:String that defines the format of date values in the data files to be loaded. 
    * **ORC**
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **AVRO**
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
    * **PARQUET**
      * **Replace invalid characters** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.

<h4 id="external-tables-additional-options"> External Tables Additional Options</h4>

* **Auto refresh**:True/False toggle that specifies whether Snowflake should enable triggering automatic refreshes of the external table metadata when new or updated data files are available in the named external stage.
  * False - Auto refresh does not take place
  * True - Auto refresh takes place. Setting this to true, give you the option to choose a Cloud Provider from AWS, Azure, and GCP.\
    * AWS - AWS SNS Topic, Enabled only when Cloud Provider is AWS and auto refresh is true.Required only when configuring AUTO_REFRESH for Amazon S3 stages using Amazon Simple Notification Service (SNS).
    * GCP - Integration. Enabled when Cloud Provider is GCP/Azure. Specifies the existing notification integration used to access the storage queue.
    * Azure - Integration. Enabled when Cloud Provider is GCP/Azure. Specifies the existing notification integration used to access the storage queue.


### System Columns
The set of columns which has source data and file metadata information.

* **VALUE** - The data from the file is loaded into this variant column
* **FILENAME** - Name of the staged data file the current row belongs to. Includes the full path to the data file.

### Key points to use CopyInto node
* External table node can be created by just clicking on Create node from browser if we want the data from the file to be loaded into single variant column in target table.
* External node can be added on top of an inferred table(table created by running the inferschema node) if you want to load data into specific columns as defined in the files.Refer to Inferschema to know more on how to use the node and add External node on top of it.

### External table Deployment

### Initial Deployment

When deployed for the first time into an environment the External table node will execute the below stage:

* **Create External Table** - This will execute a CREATE OR REPLACE statement and create a table in the target environment.

### Redeployment

#### Recreating the External table

The subsequent deployment of External table with changes in config options will recreate the table.

The following stages are executed:

* **Create External Table**
  
### Undeployment

If the External table node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

* **Drop External Table**

<h2 id="CopyUnload">CopyUnload </h2>

The Coalesce Copy Unload node unloads the table into internal stage or external stage location in various file formats supported by Snowflake

A [Copy Unload](https://docs.snowflake.com/en/sql-reference/sql/copy-into-location#examples) node loads data from a table (or query) into one or more files in the internal or external location mentioned

### Copy Unload Node Configuration

The Copy Unload node type has four configuration groups:

* [Node Properties](#copy-unload-node-properties)
* [File Location](#copy-unload-file-location)
* [File Format](#copy-unload-file-format)
* [Copy Options](#copy-unload-copy-options)

<h4 id="copy-unload-node-properties"> Copy Unload Node Properties</h4>

* **Storage Location**: Storage Location is the location(i.e. Database and Schema) of your source table. 
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="copy-unload-file-location"> Copy Unload Node File Location </h4>

![copy-unload-file](https://github.com/coalesceio/External-Data-Package/assets/169126315/02698cc6-1d16-4e0c-b0fa-c8d614a6e2ed)

* **Stage (Required)**: Location in Snowflake(internal stage) or external stage or external location where the data files are unloaded.
* **Partition by (Optional)**: Unload operation splits the table rows based on the partition expression and determines the number of files to create based on the number of unique values in a particular column (only sinlge column name expected)
* **Allow Expressions in partition by clause**:A regular expression pattern string, enclosed in single quotes, for example: ('date=' || to_varchar(dt, 'YYYY-MM-DD') || '/hour=' || to_varchar(date_part(hour, ts))- Concatenates labels and column values to output meaningful filenames

<h4 id="copy-unload-file-format">Copy Unload  File Format</h4>

<img width="217" alt="FileFormat" src="https://github.com/coalesceio/External-Data-Package/assets/169126315/53434835-c071-4132-b1f0-d0c41810216f">

* **File Format Definition - File Format Name**:
  * **File Format Name**: Specifies an existing named file format in Snowflake to use for unloading data into the table.
  * **File Type**:
    * CSV
    * JSON
    * PARQUET
* **File Format Definition - File Format Values**: 
  * **File Format Values** -Provides different file format options for the File Type chosen to unload.
  * **File Type**: Each file type has different configurations available.
    * **CSV**
        * **Compression**: String (constant) that specifies the current compression algorithm for the data files to be unloaded.
        * **Record delimiter**: Characters that separate records in an unloaded file
        * **Field delimiter**: One or more singlebyte or multibyte characters that separate fields in an unloaded file
        * **Field optionally enclosed by**: Character used to enclose strings. (Default is \042)
        * **File Extension**: String that specifies the extension for files unloaded to a stage. Accepts any extension.
        * **Date Format**: String that defines the format of date values in the unloaded data files.(Default is AUTO)
        * **Time Format**: String that defines the format of time values in the unloaded data files.(Default is AUTO)
        * **Timestamp Format**: String that defines the format of timestamp values in the unloaded data files.(Default is AUTO)
    * **JSON**
        * **Compression**: String (constant) that specifies the current compression algorithm for the data files to be unloaded.
        * **File extension** - Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character.
     * **PARQUET**
        * **Compression**: String (constant) that specifies the current compression algorithm for the data files to be unloaded.

<h4 id="copy-unload-copy-options"> Copy Unload Copy Options</h4>

<img width="217" alt="CopyOptions" src="https://github.com/coalesceio/External-Data-Package/assets/169126315/4a1d32ec-a898-40ba-93e7-94194bcae02d">


* **Overwrite Flag**: Toogle overwrites existing files with matching names, if any, in the location where files are unloaded
* **Header**: 
* **Single File Flag**: Toggle generates a single file when true, else generates multiple files if partition by enabled
* **Max File Size (MB)**: Number (> 0) that specifies the upper size limit (in bytes) of each file to be generated in parallel per thread. Note that the actual file size and number of files unloaded are determined by the total amount of data and number of nodes available for parallel processing.
* **Include Query ID**:  When true provides unique identifier for unloaded files by including a universally unique identifier (UUID) in the filenames of unloaded data files 

* **Detailed Output**: When true includes a row for each file unloaded to the specified stage. Columns show the path and name for each file, its size, and the number of rows that were unloaded to the file.

  ### Copy Unload Deployment

  Deployment not applicable for this node

<h2 id="API">API </h2>

The Coalesce API Node loads external data which is external to snowflake with the help of EXTERNAL ACCESS INTEGRATION Name which has a network rule which allows access to external network locations using procedure handler


### API Node Configuration

The API Node Configuration type has two configuration groups:

* [Node Properties](#api-node-properties)
* [Options](#API-node-Options)

<h4 id="api-node-properties"> API Node Properties</h4>

* **Storage Location**: Storage Location is the location of the destination where retrieved data of API calls are stored 
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="api-node-options"> API Node Options </h4>

<img width="235" alt="Options" src="https://github.com/coalesceio/External-Data-Package/assets/169126315/07bac308-b260-4281-82e7-1836e6fc0530">

* **Snowflake EXTERNAL ACCESS INTEGRATION Name (Required)**: EXTERNAL ACCESS INTEGRATION Name has a network rule which allows access to external network locations external to snowflake using procedure 
* **Method(Required)**: HTTP Request methods Get,Put,Post
  * **Get**: API Call to retrieve data
  * **Put**: API Call to update existing data
  * **Post**: API Call to create new data
  
* **URI(Required)**: Uniform Resoure Identifier to be provided to locate and interact with resources within a specific API
* **Headers**: headers are used to provide additional context and information about the request
* **Payload**: when making HTTP requests to a URI, the request may include a payload (also known as a body)
* ***Note:***: Payload applicable only to Put & Post API method calls
  
<h2 id="JDBC LOAD">JDBC LOAD</h2>

The Coalesce JDBC Load Node allows to connect, interact, and retrieve data from various database management systems into Snowflake

### JDBC Load Node Configuration

The JDBC Load Node Configuration type has two configuration groups:

* [Node Properties](#JDBC-load-node-properties)
* [Options](#JDBC-load-Options)

<h4 id="JDBC-load-node-properties"> JDBC Node Properties</h4>

* **Storage Location**: Storage Location is the location of the destination where retrieved from external database are stored 
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="JDBC-load-options"> JDBC Load Options </h4>

<img width="243" alt="JDBC_load_options" src="https://github.com/coalesceio/External-Data-Package/assets/169126315/5af13d5b-576a-44ce-ae5b-d02d7501cc7f">

* **JDBC Driver(Required)**: Required to establish the connection to external databse
* **JAR file external stage location(Required)**: Location of the external stage where JAR file is located 
* **JAR file name(Required)**: Name of JAR file located in external stage
* **EXTERNAL ACCESS INTEGRATION(Required)**: EXTERNAL ACCESS INTEGRATION Name has a network rule which allows access to external network locations external to snowflake using procedure 
* **External database credentials**: Username, Password of the external Database in the form snowflake secret object
* **JDBC URL(Required)**: URL of the database with port number (ex: "sqlserver-tests.database.windows.net:1433" )
* **SQL(Required)**: SQL query to retrieve the required data
* **Truncate Before**: 


<h2 id="parse-excel">Parse Excel </h2>

The Coalesce Parse Excel node parses an large excel file in stage location to json format on the target location

### Parse Excel Node Configuration

The Parse Excel node type has three configuration groups:

* [Node Properties](#parse-excel-node-properties)
* [Stored Procedure Options](#parse-excel-stored-procedure-options)
* [Excel File Processing Options](#parse-excel-file-processing-options)

<h4 id="parse-excel-node-properties"> Parse Excel Node Properties</h4>

* **Storage Location**: Storage Location is the location of the destination where source table is parsed. 
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="parse-excel-stored-procedure-options"> Parse Excel Stored Procedure Options </h4>

<img width="228" alt="StoredProcedureOptions" src="https://github.com/coalesceio/External-Data-Package/assets/169126315/ad7e21b6-1ea2-43c9-875a-5a1ce34c9828">

* **Stored Procedure Storage Location(Required)**: Location in Snowflake to be provided where Stored Procedure will be loaded by script
* ***Note***: Stored Procedure location and Storage Location (i.e. destination) must be in same Schema

<h4 id="parse-excel-file-processing-options">Excel File Processing Options</h4>

<img width="227" alt="ExcelFileProcessingOptions" src="https://github.com/coalesceio/External-Data-Package/assets/169126315/f612a1cb-f5fe-420f-9a38-d040f0151e6a">


* **Source File Storage Location(REQUIRED)**: Storage location name in Coalesce which points to the database and schema  where stage is located (source file has to be located under stages)
* **Source File Stage(REQUIRED)**: Stage name in Snowflake where source file is located
* **Source File Name(REQUIRED)**: Excel filename to be parsed

### Initial Deployment

When deployed for the first time into an environment the Parse Excel node will execute the below stage:

* **Create OR Replace Table** - This will execute a CREATE OR REPLACE statement and create a table in the target environment.
* **Create OR Replace Stored Procedure** - This will create a Stored Procedure in the target environment 

### Redeployment

#### Recreating the Parse Excel

There are few column or table changes like Change in table name, Dropping existing column, Alter Column data type, Adding a new column if made in isolation or all-together will result in an ALTER statement to modify the target Table in the target environment.Any table level changes or node config changes results in  recreation of Stored Procedure

The following stages are executed:

* **Rename Table| Alter Column | Delete Column | Add Column| Edit table description** -  Alter table statement is executed to perform the alter operation.
* **Create OR Replace Stored Procedure** - This will create a Stored Procedure in the specified target environment  
  
### Undeployment

If the Parse Excel node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

* **Drop Table**
* **Drop Procedure**

<h2 id="parse-json">Parse Json </h2>

The Coalesce parses large json file containing json array to the provided target location 

### Parse Json Node Configuration

The Parse Excel node type has three configuration groups:

* [Node Properties](#parse-json-node-properties)
* [UDTF Procedure Options](#parse-json-UDTF-procedure-options)
* [JSON File Processing Options](#parse-json-json-file-processing-options)

<h4 id="parse-json-node-properties"> Parse json Node Properties</h4>

* **Storage Location**: Storage Location is the location of the destination where source table is parsed to. 
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h4 id="parse-json-UDTF-procedure-options"> Parse JSON UDTF Procedure Options </h4>

<img width="223" alt="UDTFProcedureOptions" src="https://github.com/coalesceio/External-Data-Package/assets/169126315/c319c9f4-6654-4c9b-9416-5a979279055f">

* **User Defined Table Function Location (Required)**: Location in Snowflake to be provided where UDTF will be loaded by script
* ***Note***: UDTF location and Storage Location (i.e. destination) must be in same Schema

<h4 id="parse-json-json-file-processing-options">Parse JSON JSON File Processing Options</h4>

<img width="227" alt="JSONFileProcessingOptions" src="https://github.com/coalesceio/External-Data-Package/assets/169126315/b32ac841-6e29-4442-8474-8385cd9a0e97">


* **Source File Storage Location(REQUIRED)**: Schema name in Snowflake where source file is located (source file has to be located under stages)
* **Source File Stage(REQUIRED)**: Stage name in Snowflake where source file is located
* **Source File Name(REQUIRED)**: Json filename to be parsed
* **JSON Array Name(REQUIRED)**: Array name of the json

### Initial Deployment

When deployed for the first time into an environment the Parse Json node will execute the below stage:

* **Create OR Replace Table** - This will execute a CREATE OR REPLACE statement and create a table in the target environment.
* **Create OR Replace Function** - This will create a UDTF function in the target environment 

### Redeployment

#### Recreating the Parse Json

There are few column or table changes like Change in table name, Dropping existing column, Alter Column data type, Adding a new column if made in isolation or all-together will result in an ALTER statement to modify the target Table in the target environment.Any table level changes or node config changes results in  recreation of Stored Procedure

The following stages are executed:

* **Rename Table| Alter Column | Delete Column | Add Column| Edit table description** -  Alter table statement is executed to perform the alter operation.
* **Create OR Replace Stored Procedure** - This will create a Stored Procedure in the specified target environment  
  
### Undeployment

If the Parse Json node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

* **Drop Table**
* **Drop Procedure**
  
## Code

### InferSchema

* [Node definition](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/Inferschema-299/definition.yml)
* [Create Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/Inferschema-299/create.sql.j2)
* [Run Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/Inferschema-299/run.sql.j2)

### CopyInto

* [Node definition](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/CopyInto-324/)
* [Create Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/CopyInto-324/)
* [Run Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/CopyInto-324/)

### Snowpipe

* [Node definition](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/Snowpipe-323/)
* [Create Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/Snowpipe-323/)
* [Run Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/Snowpipe-323/)

### External Tables 

* [Node definition](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/ExternalTable-298/definition.yml)
* [Create Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/ExternalTable-298/create.sql.j2)
* [Run Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/ExternalTable-298/run.sql.j2)

### Copy Unload

* [Node definition](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/CopyUnload-302)
* [Create Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/CopyUnload-302)
* [Run Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/CopyUnload-302)

### API

* [Node definition](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/API-303)
* [Create Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/API-303)
* [Run Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/API-303)

### JDBC

* [Node definition](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/JDBCLoad-327)
* [Create Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/JDBCLoad-327)
* [Run Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/JDBCLoad-327)

### Parse Excel

* [Node definition](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/ParseExcel-346)
* [Create Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/ParseExcel-346)
* [Run Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/ParseExcel-346)

### Parse Json

* [Node definition](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/ParseJSON-347)
* [Create Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/ParseJSON-347)
* [Run Template](https://github.com/coalesceio/External-Data-Package/tree/86f1d8f019493de644094a2139a8e1a3a0a510be/nodeTypes/ParseJSON-347)

  
