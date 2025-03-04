# How-to Workshop Guide
## OPTIONAL
To create the **custom skills with Azure Function leveraging om document intelligence**, follow the training in here\
[Build a Document Intelligence custom skill for Azure AI Search - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/build-form-recognizer-custom-skill-for-azure-cognitive-search/)

## Create Azure Resources
> Note: If you have previously created an Azure Storage Account and Azure AI Search resource, and still have these Azure resources in your subscription, you can skip this section and start at the [Workshop Steps](#workshop-steps) section. Otherwise, follow the steps below to provision the required Azure resources.

1. In a web browser, open the Azure portal at https://portal.azure.com, and sign in using the Microsoft account associated with your Azure subscription.

2. In the top search bar, search for storage accounts, and create a new storage account resource with the following settings:
    - Subscription: _Your Azure subscription_
    - Resource group: _Choose or create a resource group (if you are using a restricted subscription, you may not have permission to create a new resource group - use the one provided)_
    - Storage Account Name: _Enter a unique name_
    - Region: _Choose from available regions geographically close to you_
    - Primary Service: Azure Blob Storage or Azure Data Lake Storage Gen 2
    - Performance: Standard
    - Redundancy: Locally-redundant storage (LRS)
    - Pricing tier: Standard S0
    - Accept the default values for the other settings
      
3. Once deployed, go to the resource and on the Overview page, note the Subscription ID and Location. You will need these values, along with the name of the resource group in subsequent steps.

4. In the top search bar, search for AI Search, and create an Azure AI Search resource with the following settings:
    - Subscription: _Your Azure subscription_
    - Resource group: _Choose or create a resource group (if you are using a restricted subscription, you may not have permission to create a new resource group - use the one provided)_
    - Service Name: _Enter a unique name_
    - Location: _Choose from available regions geographically close to you_
    - Pricing tier: Free
    > **[NOTE]** You may only have 1 free AI Search resource in one subscription

5. Once deployed, go to the resource and on the Overview page, note the Subscription ID and Location. You will need these values, along with the name of the resource group in subsequent steps.


## Workshop Steps 

1. Upload sample document into Azure Blob storage.
    - Download the [sample document](https://github.com/AITechnicalReadiness/AIWorkshop-AzureAISearch/tree/main/Invoice) 
    - Upload all 5 files to Azure Blob storage
      > **[NOTE]** Make sure to **Enabled** the storage account to “Allow storage account key access” 
      ![Screenshot of enable key access.](media/enabled_keyaccess.png#lightbox)

2. Create the data source in Azure AI Search and **create connection** to the Azure Blob Storage.
    - Go to **Data sources” > “+ Add data source”** and configure to the Azure Blob Storage that contains the 5 invoices document. 
      ![Screenshot of add Datasource](media/add_datasource.png#lightbox)

3. Configuration in **Azure AI Search**
    - Include the WebApiSkill in **Skillsets**. You can get the full JSON for the skillsets in [skillsets_customskills.txt](https://github.com/AITechnicalReadiness/AIWorkshop-AzureAISearch/blob/main/skillsets_customskills.txt)
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
    - Add new index into **Indexes**. You can get the full index definition from [index_customskills.txt](https://github.com/AITechnicalReadiness/AIWorkshop-AzureAISearch/blob/main/index_customskills.txt).
      > Indexer maps the outputs from the skill to the fields in the index
      ![Screenshot of add new index](media/add_newindexes.png#lightbox)
    
    - Modify the **Indexer**. You can get the full Indexer JSON from [indexer_customskills.txt](https://github.com/AITechnicalReadiness/AIWorkshop-AzureAISearch/blob/main/indexer_customskills.txt).
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
4. Expected Outcome
   ![Screenshot of Expected Outcome](media/expected_outcome.png#lightbox)



## Additional Note
+ **Skillsets**
  -  The input to the Function App (formUrl & formSasToken)
     ![Screenshot of input function app](media/input_functionapp.png#lightbox)

  - (input) to Skillset in Azure AI Search
     ![Screenshot of input azure ai search](media/input_azureaisearch.png#lightbox)

  - The output from the Function App (invoices)
     ![Screenshot of output function app](media/output_functionapp.png#lightbox)

  - (output) Skillset in Azure AI Search
     ![Screenshot of output azure ai search](media/output_azureaisearch.png#lightbox)

+ **Indexer** (partial)
  + outputFieldMappings :
     ![Screenshot of indexer](media/indexer.png#lightbox)
+ **Index**
  ![Screenshot of index](media/index.png#lightbox)
