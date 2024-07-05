# External Data Package

The Coalesce External Data Package includes:

* [Inferschema](#inferschema)
* [CopyInto](#CopyInto)
* [CopyInto-Snowpipe](#CopyIntoSnowpipe)
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

### InferSchema Example workflow

* Add a InferSchema node, for example `INFER_JSON`.
* Deploy the node. On deployment a table of the same name as the InferSchema node is created with columns inferred on parsing the file. For example, `InferSchema node:INFER_JSON,Inferred table:INFER_JSON`.
* Add the inferred table to the browser using Add Sources tab.
* Now we can add a Copy-Into Snowpipe or External table node on top of the inferred table to load staged files.
* If the structure of the file is changed, you can redeploy the InferSchema node with "Alter existing table" redeployment behaviour. The already inferred table is altered.
* Next, you can re-sync the columns of the inferred table and redeploy the CopyInto Copy-Into Snowpipe or External table node added on top of it.

> 🚧 Inferred table and Copy-into nodes/External table cannot be deployed together
>
> Infer Schema node, inferred table and Copy-into nodes/External table cannot be deployed together. First deploy the Infer Schema node. Then add the inferred table in browser, add Copy-into node on top of the table and then deploy the same.

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
* [Snowpipe Options](#copy-into-snowpipe-options)
* [File Location](#copy-into-file-location)
* [File Format](#copy-into-file-format)
* [Copy Options](#copy-into-copy-options)

<h3 id="copy-into-node-properties">CopyInto - Node Properties</h3>

* **Storage Location**: Storage Location where the WORK will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h3 id="copy-into-file-location">  CopyInto - File Location </h3>

![Filelocation](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/48b7a0dd-301c-4077-857c-e37fd234bbf8)

* **Stage Storage Location (Required)**: A storage location in Coalesce where the stage is located.
* **Stage Name (Required)**: Internal or External stage where the files containing data to be loaded are staged.
* **File Names**: Specifies a list of one or more files names (separated by commas) to be loaded. For example, `'a.csv','b.csv'`.
* **File Pattern**:A regular expression pattern string, enclosed in single quotes, specifying the file names or paths to match. For example, `*hea.*[.]csv'`.

<h3 id="copy-into-file-format"> CopyInto - File Format </h3>

![fileformat](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/82945bb5-ba5d-46dd-b66d-3d6c10ff77d9)


* **File Format Definition - File Format Name**:
  * **File Format Name**: Specifies an existing named file format to use for loading data into the table.
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


### CopyInto - Snowpipe Deployment

#### CopyInto Snowpipe Deployment Parameters

The CopyInto-Snowpipe includes an environment parameter that allows you to specify if you want to perform a full load or a reload based on the load type when you are performing a Copy-Into operation.

The parameter name is `loadType` and the default value is ``.

```json
{
    "loadType": ""
}
```

When the parameter value is set to `Reload`, the data is reloaded into the table regardless of whether they’ve been loaded previously and have not changed since they were loaded.

#### CopyInto - Snowpipe Initial Deployment

When deployed for the first time into an environment the CopyInto-Snowpipe node will execute the below stage depending on if Snowpipe is enabled and the `loadType`.

| Deployment Behavior  | Enable Snowpipe |Stages Executed|
|--|--|---|--|
|  Initial Deployment | `false` | empty|Create Table. </br> Historical full load using CopyInto. |
| Initial Deployment | `false` | `Reload` | Truncate Target Table </br> Reload data-Copy Into Force|

### CopyInto - Snowpipe Redeployment

#### Altering the CopyInto-Snowpipe Tables

There are few column or table changes like Change in table name, Dropping existing column,  Alter Column data type, Adding a new column if made in isolation or all-together will result in an ALTER statement to modify the target Table in the target environment.Any table level changes or node config changes results in recreation of pipe

The following stages are executed:

* **Rename Table| Alter Column | Delete Column | Add Column| Edit table description |**: Alter table statement is executed to perform the alter operation.
* **Create Pipe**: Pipe is recreated if enable snowpipe option is true
* **Historical full load using CopyInto**:Historical data are loaded

### CopyInto - Snowpipe Undeployment

If the CopyInto-Snowpipe node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

This is executed in two stages:

* **Drop Table**: Target table is dropped
* **Drop Pipe**: Pipe is dropped


<h2 id="CopyIntoSnowpipe">CopyInto - Snowpipe </h2>

The Coalesce CopyInto - Snowpipe node is a node that performs two operations. It can be used to load historical data using     
 [CopyInto](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table). CopyInto-Snowpipe node can be used to create a pipe to auto ingest files from AWS, GCP, or Azure.

[Snowpipe](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro) enables loading data from files as soon as they’re available in a stage. 

This means you can load data from files in micro-batches, making it available to users within minutes, rather than manually executing COPY statements on a schedule to load larger batches.

### CopyInto - Snowpipe Node Configuration

The Copy-Into Snowpipe node type the following configurations available:

* [Node Properties](#copy-into-snowpipe-node-properties)
* [Snowpipe Options](#copy-into-snowpipe-snowpipe-options)
* [File Location](#copy-into-snowpipe-file-location)
* [File Format](#copy-into-snowpipe-file-format)
* [Copy Options](#copy-into-snowpipe-copy-options)

![Snowpipe-config](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/14cfeadf-2996-4665-80a1-ecd528691326)

<h3 id="copy-into-snowpipe-node-properties">CopyInto - Snowpipe Node Properties</h3>

* **Storage Location**: Storage Location where the WORK will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

<h3 id="copy-into-snowpipe-snowpipe-options"> CopyInto - Snowpipe Options </h3>

* **Enable Snowpipe**: Toggle that helps us to load create a pipe to auto ingest files from external stage.
  * Toggle Off - Provides option to load data from files in internal or external stage. This is under File Location configuration.
  * Toggle On - Load data auto ingesting files from AWS, Azure, or GCP. Choose your **Cloud Provider.**
    * AWS - AWS SNS Topic. Specifies the Amazon Resource Name (ARN) for the SNS topic for your S3 bucket.
    * Azure - Integration. Specifies the existing notification integration used to access the storage queue.
    * GCP -  Integration. Specifies the existing notification integration used to access the storage queue.
* **Load historical data**:When the toggle is enabled auto ingest does not take place and the files already in internal or external stages are loaded

<h3 id="copy-into-snowpipe-file-location">  CopyInto - Snowpipe File Location </h3>


![Filelocation](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/48b7a0dd-301c-4077-857c-e37fd234bbf8)

* **Stage Storage Location (Required)**: A storage location in Coalesce where the stage is located.
* **Stage Name (Required)**: Internal or External stage where the files containing data to be loaded are staged.
* **File Names**: Enabled when 'Enable Snowpipe' under Snowpipe Options is toggled off. Specifies a list of one or more files names (separated by commas) to be loaded. For example, `'a.csv','b.csv'`.
* **File Pattern**:A regular expression pattern string, enclosed in single quotes, specifying the file names or paths to match. For example, `*hea.*[.]csv'`.


<h3 id="copy-into-snowpipe-file-format"> CopyInto - Snowpipe File Format </h3>

![fileformat](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/82945bb5-ba5d-46dd-b66d-3d6c10ff77d9)


* **File Format Definition - File Format Name**:
  * **File Format Name**: Specifies an existing named file format to use for loading data into the table.
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

<h3 id="copy-into-snowpipe-copy-options"> CopyInto - Snowpipe Copy Options </h3>

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

### CopyInto - Snowpipe System Columns

The set of columns which has source data and file metadata information.

* **SRC** - The data from the file is loaded into this variant column.
* **LOAD_TIMESTAMP** - Current timestamp when the file gets loaded.
* **FILENAME** - Name of the staged data file the current row belongs to. Includes the full path to the data file.
* **FILE_ROW_NUMBER** - Row number for each record in the staged data file.
* **FILE_LAST_MODIFIED** - Last modified timestamp of the staged data file the current row belongs to
* **SCAN_TIME** - Start timestamp of operation for each record in the staged data file. Returned as TIMESTAMP_LTZ.


### CopyInto - Snowpipe Deployment

#### CopyInto - Snowpipe Initial Deployment

When deployed for the first time into an environment the CopyInto-Snowpipe node will execute the below stage depending on if Snowpipe is enabled and the `loadType`.

| Deployment Behavior  | Enable Snowpipe |Stages Executed|
|--|--|---|--|
|  Initial Deployment | `false` | Create Table. </br> Historical full load using CopyInto. |
| Initial Deployment | `true` | Create table </br> Create Pipe |

### CopyInto - Snowpipe Redeployment

#### Altering the CopyInto-Snowpipe Tables

There are few column or table changes like Change in table name, Dropping existing column,  Alter Column data type, Adding a new column if made in isolation or all-together will result in an ALTER statement to modify the target Table in the target environment.Any table level changes or node config changes results in recreation of pipe

The following stages are executed:

* **Rename Table| Alter Column | Delete Column | Add Column| Edit table description |**: Alter table statement is executed to perform the alter operation.
* **Create Pipe**: Pipe is recreated if enable snowpipe option is true
* **Historical full load using CopyInto**:Historical data are loaded

### CopyInto - Snowpipe Undeployment

If the CopyInto-Snowpipe node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

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
  
## Code

### InferSchema

* [Node definition](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/Inferschema-299/definition.yml)
* [Create Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/Inferschema-299/create.sql.j2)
* [Run Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/Inferschema-299/run.sql.j2)

### CopyInto-Snowpipe

* [Node definition](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/CopyInto-Snowpipe-297/definition.yml)
* [Create Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/CopyInto-Snowpipe-297/create.sql.j2)
* [Run Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/CopyInto-Snowpipe-297/run.sql.j2)

### External Tables 

* [Node definition](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/ExternalTable-298/definition.yml)
* [Create Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/ExternalTable-298/create.sql.j2)
* [Run Template](https://github.com/coalesceio/External-Data-Package/blob/main/nodeTypes/ExternalTable-298/run.sql.j2)
