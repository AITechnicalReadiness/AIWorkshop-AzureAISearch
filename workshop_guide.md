# How-to Workshop Guide
## Step
1. `[OPTIONAL]` Follow the training in here to create the **custom skills with Azure Function leveraging om document intelligence:** [Build a Document Intelligence custom skill for Azure AI Search - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/build-form-recognizer-custom-skill-for-azure-cognitive-search/)

2. Upload sample document into Azure Blob storage.
    - Download the [sample document](https://github.com/MicrosoftLearning/mslearn-ai-document-intelligence/tree/main/Labfiles/04-custom-skill/SampleInvoices) 
    - Upload all 5 files to Azure Blob storage
      > **[NOTE]** Make sure to **Enabled** the storage account to “Allow storage account key access” 
      ![Screenshot of enable key access.](media/enabled_keyaccess.png#lightbox)

3. Create the data source in Azure AI Search and **create connection** to the Azure Blob Storage.
    - Go to **Data sources” > “+ Add data source”** and configure to the Azure Blob Storage that contains the 5 invoices document. 
      ![Screenshot of add Datasource](media/add_datasource.png#lightbox)

4. Configuration in **Azure AI Search**
    - Include the WebApiSkill in **Skillsets**. You can get the full JSON for the skillsets in **skillsets_customskills.txt**
      ```json
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "name": "formrecognizer",
        "description": "Analyze invoices and extract fields.",
        "context": "/document",
        "uri": "Azure Function URL for AnalyzeInvoice",  
        "httpMethod": "POST",
        "timeout": "PT30S",
        "batchSize": 1,
        "degreeOfParallelism": null,
        "authResourceId": null,
        "inputs": [
          {
            "name": "formUrl",
            "source": "/document/url"
          },
          {
            "name": "formSasToken",
            "source": "/document/metadata_storage_sas_token"
          }
        ],
        "outputs": [
          {
            "name": "invoices",
            "targetName": "invoices"
          }
        ],
        "httpHeaders": {},
        "authIdentity": null
      },
      ```
    - Add new index into **Indexes**. You can get the full index definition from **index_customskills.txt**. 
    <br>
      `Indexer maps the outputs from the skill to the fields in the index`
      ![Screenshot of add new index](media/add_newindexes.png#lightbox)
      - Modify the **Indexer**. You can get the full Indexer JSON from **indexer_customskills.txt**
        ```json 
        "fieldMappings": [
            {
            "sourceFieldName": "metadata_storage_path",
            "targetFieldName": "metadata_storage_path",
            "mappingFunction": {
                "name": "base64Encode",
                "parameters": null
            }
            },
            {
            "sourceFieldName": "metadata_storage_path",
            "targetFieldName": "url",
            "mappingFunction": null
            }
        ],

        "outputFieldMappings": [
        {
            "sourceFieldName": "/document/invoices",
            "targetFieldName": "invoices"
            },
            {
            "sourceFieldName": "/document/invoices/*/CustomerAddress",
            "targetFieldName": "address"
            },
            {
            "sourceFieldName": "/document/invoices/*/InvoiceId",
            "targetFieldName": "invoiceId",
            "mappingFunction": null
            }
        ],
        ```
5. Expected Outcome
   ![Screenshot of Expected Outcome](media/expected_outcome.png#lightbox)


## Additional Note
+ **Skillsets**
  1. The input to the Function App (formUrl & formSasToken)
     ![Screenshot of input function app](media/input_functionapp.png#lightbox)
  2. (input) to Skillset in Azure AI Search
     ![Screenshot of input azure ai search](media/input_azureaisearch.png#lightbox)
  3. The output from the Function App (invoices)
     ![Screenshot of output function app](media/output_functionapp.png#lightbox)
  4. (output) Skillset in Azure AI Search
     ![Screenshot of output azure ai search](media/output_azureaisearch.png#lightbox)

+ **Indexer** (partial)
  + outputFieldMappings :
     ![Screenshot of indexer](media/indexer.png#lightbox)
+ **Index**
  ![Screenshot of index](media/index.png#lightbox)