# External Data Package

 * [InferSchema](#inferschema)
 * [CopyInto Snowpipe](#copyinto-snowpipe)
 * [External Tables](#external-tables)
 * [Code](#code)


# InferSchema

The Coalesce InferSchema UDN is a versatile node infers schema of the file in internal or external stage and dynamically creates the target table of the same name as inferschema node.

[InferSchema](https://docs.snowflake.com/en/sql-reference/functions/infer_schema) in Snowflake automatically detects the file metadata schema in a set of staged data files that contain semi-structured data and retrieves the column definitions.

## Version Information
 * UDN Name: InferSchema
 * Package Name: 

 * Certified Version: 1.0
 * Documentation Version: 1.0
 * Current Version: 1.1
 * Originally Developed by: Anandhi
 * Current Owner: Anandhi
 * Materialization Type: table
 * Create Template: Yes
 * Run Template: No

 * Deployment Strategy: Advanced
 * Deployment Method: ALTER when possible otherwise CREATE OR REPLACE

 * Test Enabled: No

## Node Configuration

The InferSchema node type has two configuration groups:

* [Node Properties](#node-properties)
* [Source data](#source-data)

![nodeinferschema-config](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/cb9a8008-60e0-435a-b0f8-80bc13c58a3c)

### Node Properties

There are four configs within the **Node Properties** group.

* **Storage Location**: Storage Location where the WORK will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.
  
### Source Data

![Inferschema-Source Data](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/f452992d-0ce0-4bb9-8ece-857cb52e745b)

There are many configs within the **Source Data** group.

* **Stage Storage Location**:A storage location in coleasce wherein the stage is located.
* **Stage Name**: Internal or External stage where the files containing data to be loaded are staged
* **File Name(s) ( Use commas to seperate multiple files)**: File whose metatdata is to be inferred
* **File Format Name**: Name of the file format object that describes the data contained in the staged files.It is expected in the same location as Stage.
* **Redeployment Behavior**: 
    * **CREATE OR REPLACE**: Dynamically creates target table based on the inferred schema from file in staging area.
    * **ALTER EXISTING TABLE**: Dynamically alters inferred table by comparing the inferred schema of the same file (with changes if any)and created table 
	* **DROP EXISTING TABLE**: Drops the table inferred
	
### Prerequisites

* **A sample file in the internal or external stage**
* **An existing fileformat for the specific file type**

## Example workflow

* Add a InferSchema node(ex:INFER_JSON)
* Deploy the node.On deployment ,a table of the same name as InferSchema node is created with columns inferred on parsing the file.(Ex:InferSchema node:INFER_JSON,Inferred table:INFER_JSON)
* Add the inferred table to the browser using Add Sources tab.
* Now we can add a Copy-Into Snowpipe or External table node on top of the inferred table to load staged files.
* If the structure of the file is changed,you can redeploy the InferSchema node with "Alter existing table" redeployment behaviour.The already inferred table is altered.
* Next,you can re-sync the columns of the inferred table and redeploy the CopyInto Copy-Into Snowpipe or External table node added on top of it.

*Note: Infer Schema node, inferred table and Copy-into nodes/External table cannot be deployed together. First deploy the Infer Schema node. Then add the inferred table in browser, add Copy-into node on top of the table and then deploy the same.

### Deployment

### Initial Deployment

When deployed for the first time into an environment the InferSchema node will execute the below stage

* **Infer and Create target table**

### Redeployment
 
Redeployment behavior-Create or Replace

If any of the Source Data options like Stage storage location, Stage name or filename are modified.
 Then we can redeploy the Infer Schema node with redeployment behaviour “Create or Replace”.

The following stage is executed:

* **Infer and Create target table**
	 
Tip: Next you can go back to the browser and Re-sync columns of Inferred table, re-execute Copy-Into and redeploy

Redeployment behavior-Alter existing table

If all Source Data options remain same and only there are changes in the existing file structure, 
we can redeploy the Infer Schema node with redeployment behaviour “Alter existing table”.

The following stage is executed:
    
 * **Infer and Alter target table**   
	 
Redeployment behavior-Drop existing table

  If you want to drop the inferred table you can redeploy the Infer Schema node with redeployment behaviour “Drop existing table”
  
* **Drop inferred table** 
  
### Undeployment

If the InferSchema node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then no action takes place.


# CopyInto Snowpipe

The Coalesce CopyInto-Snowpipe node is a node that performs two operations.It can be used to load historical data using CopyInto.
Also can be used to create a pipe to auto ingest files from AWS,GCP or Azure.

[Snowpipe](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro) enables loading data from files as soon as they’re available in a stage. 
This means you can load data from files in micro-batches, making it available to users within minutes, rather than manually executing COPY statements on a schedule to load larger batches.

## Version Information
 * UDN Name: InferSchema
 * Package Name: 

 * Certified Version: 1.0
 * Documentation Version: 1.0
 * Current Version: 1.1
 * Originally Developed by: John Gontarz
 * Current Owner: John Gontarz
 * Materialization Type: table
 * Create Template: Yes
 * Run Template: No

 * Deployment Strategy: Advanced
 * Deployment Method: ALTER when possible otherwise CREATE OR REPLACE

 * Test Enabled: No
 
 ## Node Configuration

The InferSchema node type has two configuration groups:

* [Node Properties](#node-properties)
* [Snowpipe Options](#snowpipe-options)
* [File Location](#file-location)
* [File Format](#file-format)
* [Copy Options](#copy-options)

![Snowpipe-config](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/14cfeadf-2996-4665-80a1-ecd528691326)

### Node Properties

There are four configs within the **Node Properties** group.

* **Storage Location**: Storage Location where the WORK will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

### Snowpipe Options

* **Enable Snowpipe**:True/False toggle that helps us to load historical data or create a pipe to auto ingest files from external stage
       * True - provides option to load data from files in internal or external stage
      * False - provides option to load data by auto ingesting files from external stage/location in AWS,GCP,Azure
* **Cloud Provider**: 
      * AWS
	  * GCP
	  * Azure
* **AWS SNS Topic**: Enabled only when Cloud Provider is AWS.Specifies the Amazon Resource Name (ARN) for the SNS topic for your S3 bucket.
* **Integration**:Enabled when Cloud Provider is GCP/Azure.Specifies the existing notification integration used to access the storage queue.

![Snowpipe](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/49b17d06-2a35-4bf4-b208-145e5f9c607d)

### File Location

* **Stage Storage Location**:A storage location in coleasce wherein the stage is located.
* **Stage Name**: Internal or External stage where the files containing data to be loaded are staged
* **File Name(s) (Optional - Ex:'a.csv','b.csv' ))**: Enabled when 'Enable Snowpipe' option is disabled.Specifies a list of one or more files names (separated by commas) to be loaded
* **File Pattern(Optional - Ex:'.*hea.*[.]csv')**:A regular expression pattern string, enclosed in single quotes, specifying the file names and/or paths to match.

![Filelocation](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/48b7a0dd-301c-4077-857c-e37fd234bbf8)

### File Format

![fileformat](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/82945bb5-ba5d-46dd-b66d-3d6c10ff77d9)


* **File Format Definition**:Specifies the format of the data files to load.
    *File Format Name -Specifies an existing named file format to use for loading data into the table
	*File Format Values -Provides file format options for thr File Type chosen
* **File Type**:Specifies the type of files to load into the table
    *CSV
	*JSON
	*ORC
	*AVRO
	*PARQUET
	*XML

File Type -CSV /JSON/PARQUET/AVRO/ORC/XML

* **Replace invalid characters**:Boolean that specifies whether to replace invalid UTF-8 characters with the Unicode replacement character

File Type -CSV /JSON/PARQUET/AVRO/ORC

* **Compression**:String (constant) that specifies the current compression algorithm for the data files to be loaded.
* **Trim Space**:Boolean that specifies whether to remove white space from fields.

File Type -CSV

* **Compression**:String (constant) that specifies the current compression algorithm for the data files to be loaded.
* **Record delimiter**:Characters that separate records in an input file
* **Field delimiter**:One or more singlebyte or multibyte characters that separate fields in an input file
* **Field optionally enclosed by**:Character used to enclose strings
* **Number of header lines to skip**:Number of lines at the start of the file to skip.
* **Skip blank lines**:Boolean that specifies to skip any blank lines encountered in the data files
* **Date format**:String that defines the format of date values in the data files to be loaded. 
* **Time format**:String that defines the format of time values in the data files to be loaded
* **Timestamp format**:String that defines the format of timestamp values in the data files to be loaded.

File Type -JSON

* **Strip Outer Array**:
* **Date format**:String that defines the format of date values in the data files to be loaded. 
* **Time format**:String that defines the format of time values in the data files to be loaded
* **Timestamp format**:String that defines the format of timestamp values in the data files to be loaded.

### Copy Options

Enable Snowpipe-False

![copyintooptions2](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/a2dff298-801b-4b29-b303-0a295b82322c)

* **On Error Behavior**:String (constant) that specifies the error handling for the load operation.
    *CONTINUE
	*SKIP_FILE
	*SKIP_FILE_num
	*SKIP_FILE_num%
        *ABORT_STATEMENT
* ** Specify the number of errors that can be skipped**:When the **On Error Behavior** is either 'SKIP_FILE_num' or 'SKIP_FILE_num%',
 the number of errorsis required which will replace 'num' value.
* **Size Limit**:Number (> 0) that specifies the maximum size (in bytes) of data to be loaded for a given COPY statement
* **Purge Behavior**:Boolean that specifies whether to remove the data files from the stage automatically after the data is loaded successfully.
* **Return Failed Only**:Boolean that specifies whether to return only files that have failed to load in the statement result.
* **Force**:Boolean that specifies to load all files, regardless of whether they’ve been loaded previously and have not changed since they were loaded. 
* **Load Uncertain Files**:Boolean that specifies to load files for which the load status is unknown. The COPY command skips these files by default.
* **Enforce Length**:Boolean that specifies whether to truncate text strings that exceed the target column length
* **Truncate Columns**:Boolean that specifies whether to truncate text strings that exceed the target column length

Enable Snowpipe-True 

![copyoptions2](https://github.com/prj2001-udn/External-Data-Package/assets/169126315/6de13884-7e02-42c0-a79d-7836ce622c2a)

* **On Error Behavior**:String (constant) that specifies the error handling for the load operation.
    *CONTINUE
	*SKIP_FILE
	*SKIP_FILE_num
	*SKIP_FILE_num%
* **Specify the number of errors that can be skipped**:When the **On Error Behavior** is either 'SKIP_FILE_num' or 'SKIP_FILE_num%',
 the number of errorsis required which will replace 'num' value.
* **Enforce Length**:Boolean that specifies whether to truncate text strings that exceed the target column length
* **Truncate Columns**:Boolean that specifies whether to truncate text strings that exceed the target column length

### System Columns
The set of columns which has source data and file metadata information

* **SRC**
* **LOAD_TIMESTAMP**
* **FILENAME**
* **FILE_ROW_NUMBER**
* **FILE_LAST_MODIFIED**
* **SCAN_TIME**

#### SRC
The data from the file is loaded into this variant column

#### LOAD_TIMESTAMP
Current timestamp when the file gets loaded

#### FILENAME
Name of the staged data file the current row belongs to. Includes the full path to the data file.

#### FILE_ROW_NUMBER
Row number for each record in the staged data file.

#### SCAN_TIME
Start timestamp of operation for each record in the staged data file. Returned as TIMESTAMP_LTZ.

## Deployment

### Deployment Parameters

The CopyInto-Snowpipe includes an environment parameter that allows you to specify if you want to perform a full load or a reload based on the load type when you are performing a Copy-Into operation

The parameter name is `loadType` and the default value is ``.


```json
{
    "loadType": "Reload"
}
```

When the parameter value is set to `Reload`,the data is reloaded into the table regardless of whether they’ve been loaded previously and have not changed since they were loaded. 

### Initial Deployment

When deployed for the first time into an environment the InferSchema node will execute the below stage with Enable Snowpipe config set to false and paremeter loadType is empty

* **Create table**
* **Historical Full load-Using CopyInto**

When deployed for the first time into an environment the InferSchema node will execute the below stage with Enable Snowpipe config set to false and paremeter loadType is set to Reload

* **Truncate Target Table**
* **Reload data-Copy Into Force**
  
When deployed for the first time into an environment the InferSchema node will execute the below stage with Enable Snowpipe config set to true

* **Create table**
* **Create Pipe**
  
### Redeployment

After the CopyInto-Snowpipe node with materialization type table has been deployed for the first time into a target environment, subsequent deployments may result in either altering the WORK Table or recreating the WORK table.

#### Altering the CopyInto-Snowpipe tables

There are few column or table changes like Change in table name,Dropping existing column, Alter Column data type,Adding a new column if made in isolation or all-together will result in an ALTER statement to modify the target Table in the target environment.

The following stages are executed:

* **Clone Table**: Creates an internal table.
* **Rename Table| Alter Column | Delete Column | Add Column| Edit table description |**: Alter table statement is executed to perform the alter operation.
* **Swap cloned Table**: Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost.
* **Delete Table**: Drops the internal table.

### Undeployment

If the CopyInto-Snowpipe node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the target table in the target environment will be dropped.

This is executed in two stages:

* **Delete Table: Coalesce Internal table is dropped**
* **Delete Table: Target table in Snowflake is dropped**

# External Tables

The Coalesce External Table nodes create a new external table in the current/specified schema or replaces an existing external table.

An [external table](https://docs.snowflake.com/en/sql-reference/sql/create-external-table#examples) reads data from a set of one or more files in a specified external stage which can point to AWS,GCP or Azure cloud providers.


