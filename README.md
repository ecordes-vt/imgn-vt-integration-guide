
# Imagn - Veritone Integration Guide

Sections:

[creating a job](#creating-a-job)
  
  - [facial recognition](#facial-recognition)
  
  - [logo recognition](#logo-recognition)
  
  - [object recognition](#object-recognition)
  
[checking job status](#checking-job-status)

[retrieving job output](#retrieving-job-output)

[job notifications](#job-notifications)

## Veritone API Flow

1. `createJob` - Create a cognition job.
2. `checkJobStatus` - Check the status of a job. (optional, you can also use a webhook)
3. `retrieveEngineOutput` - Retrieve the output of a completed job.

## Using Veritone's API

See Veritione's [GraphQL API schema](https://api.veritone.com/v3/graphqldocs/) for a technical look at each mutation or query.

### Set your request headers

Setting a header "x-veritone-application":"org:orgGuid" allows Veritone to monitor API usage and provides better tracking for debugging purposes.  

Example: `"x-veritone-application": "org:your_org_guid"`

Please set `x-veritone-application` header on all API requests. If you do not know your orgGuid, contact your customer service manager. 


## Example Veritone API Calls

# Creating a job

Choose a template from the options below that fits your desired job type.  There are two necessary items for each job.

- `sourceUrl` A link to your file (signedUrl).

- `engineId` Id of your desired cognition engine.

For a more in-depth look at Job creation, visit the official [Veritone documentation](https://docs.veritone.com/#/quickstart/jobs/?id=working-with-jobs).

# Facial Recognition

Facial Recognition engineId(s):

 Engine Name             | engineId                             | payload name
 ----------------------- | ------------------------------------ | --------------------------
 Facebox Recognize  | e62665c7-f855-4168-8aa3-668a7b0a50ea | libraryId, libraryEngineModelId, mode
 Amazon Recognition  | df6e189f-8947-4c73-a30b-f786defc60e8 | libraryId, libraryEngineModelId, mode
 
NOTE: For testing purposes here is our default celebrity library to use.  You will have a custom library and model ID once it is has been built and trained.

Engine Name              | libraryId                            | libraryEngineModelId
 ----------------------- | ------------------------------------ | --------------------------
 Facebox Recognize  | 13e6f4a3-0d5c-4e11-9a30-913e981cb9ad | 5a49ed1b-7d1e-426c-8f78-bacc1d537ffe
 Amazon Recognition   | 13e6f4a3-0d5c-4e11-9a30-913e981cb9ad | fd637d9f-4168-421f-a81a-664ae7b41c5e

### Facial Recogntion template

This mutation returns a jobId and a TDOId.  These values are used when retrieving information about the job.  The TDOId holds all the information on a file and can have multiple jobs associated with it.  When you first create a job, a jobId and TDOId will be generated for you.  Subsequent job requests on the same file can be made to the TDOId.

```
mutation facialRecognitionJob{
    createJob(input: {
      target: { status: "downloaded" } 
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "rawchunk",
            url: "FILE_URL"
          }
          ioFolders: [
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: false, priority:-20 }
        }
        {
          engineId: "ENGINE_ID"
          payload: {
            mode: "library-run"
            libraryId: "LIBRARY_ID"
            libraryEngineModelId: "MODEL_ID"
          }        
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          # Output writer
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      targetId
      id
}}
```

# Logo Recognition

Logo Recognition engineId(s):

 Engine Name             | engineId                             | payload name
 ----------------------- | ------------------------------------ | --------------------------
 LogoGrab (low res)      | 4a52a78f-3bc7-4b3a-9b5a-1a192cc6cfe1 | key


```
mutation logoRecognitionJob {
    createJob(input: {
      target: { status: "downloaded" } 
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "rawchunk"
            url: "LINK_TO_YOUR_FILE"
          }
          ioFolders: [
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: false, priority:-20 }
        }
        {
          engineId: "ENGINE_ID"
          payload: { 
            key: "1arju7us157v6cldj6u6bdd7bnhg5o8095dru7nh" 
          }        
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          # Output writer
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      targetId
      id
}}
```

# Object Recognition

Object Recognition engineId(s):

 Engine Name             | engineId                             | payload name
 ----------------------- | ------------------------------------ | --------------------------
 Machinebox Tagbox  | d66f553d-3cef-4c5a-9b66-3e551cc48b4b | n/a

```
mutation objectRecognitionJob{
    createJob(input: {
      target: { status: "downloaded" } 
      tasks: [
        {
          engineId: "8bdb0e3b-ff28-4f6e-a3ba-887bd06e6440"
          payload:{
            ffmpegTemplate: "rawchunk",
            url: "FILE_URL"
          }
          ioFolders: [
            { referenceId: "siOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: false, priority:-20 }
        }
        {
          engineId: "ENGINE_ID"  
          ioFolders: [
            { referenceId: "engineInputFolder", mode: chunk, type: input }
            { referenceId: "engineOutputFolder", mode: chunk, type: output }
          ]
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
        }
        {
          # Output writer
          engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
          executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
          ioFolders: [
            { referenceId: "owInputFolder", mode: chunk, type: input }
          ]
        }
      ]
      routes: [
        { 
          parentIoFolderReferenceId: "siOutputFolder"
          childIoFolderReferenceId: "engineInputFolder"
          options: {}
        }
        { 
          parentIoFolderReferenceId: "engineOutputFolder"
          childIoFolderReferenceId: "owInputFolder"
          options: {}
        }
      ]
    }){
      targetId
      id
}}
```

# Checking job status

You can poll for job status updates or use webhook notifications. (detailed below)

```
query queryJobStatus {
  job(id:"JOB_ID") {
    id
    status
    targetId
    tasks{
      records{
        id
        status
        engine{
          id
          name
        }
        # taskOutput - include this field for a more verbose output
      }
    }
  }
}
```
# Retrieving job output

Include the TDO ID and Engine ID to retrieve transcription output in JSON format.

```
query getEngineOutput {
  engineResults(tdoId: "TDO_ID",
    engineIds: ["ENGINE_ID"]) { # Retrieves results from specified engine
    records {
      tdoId
      engineId
      startOffsetMs
      stopOffsetMs
      jsondata
      assetId
      userEdited
}}}
```
# Job Notifications

Veritone's `notificationUris` field allows you to link to a custom endpoint(s).  This removes the need to poll for job completion, you will instead be notified when a job or task completes at the endpoint provided.

This example notifies you when the output is completed:

```
{
  # Output writer
  engineId: "8eccf9cc-6b6d-4d7d-8cb3-7ebf4950c5f3"
  executionPreferences: { parentCompleteBeforeStarting: true, priority:-20 }
  ioFolders: [
    { referenceId: "owInputFolder", mode: chunk, type: input }
  ]
  notificationUris: ["https://example.net/hook"]
}
```
