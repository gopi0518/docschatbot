# genai-docsbot

A web portal that enables a GenAI chatbot experience on PDF documents allows users to interact with their documents through a generative AI-powered chatbot. This experience typically includes the following features:

1. **Data augmentation**:
   
   * Users can upload financial and account summary documents in PDF format.
   * The platform processes these documents by dividing them into pages and publishing each page's content, along with relevant metadata, to a Confluent Kafka topic.
   * A fully managed Confluent Flink service is then used to create vector representations of the document data for more efficient AI interactions and queries.

   Data Augmentation flow:
![Data Augmentation flow](img/DataAugmentation.jpeg)



3. **AI-Powered Interaction**: The portal is integrated with a generative AI model (like GPT) that can read, understand, and interact with the content of the documents. Users can ask the chatbot questions related to the document, request summaries, seek clarifications, or ask for specific sections or details. The AI can generate responses based on the content of the documents.
   
   Data Inference flow:
![Data Inference flow](img/DataInference.jpeg)
   

5. **Contextual Understanding**: The chatbot can understand the context of questions in relation to the document's content, making the interaction more meaningful and accurate. It can pull information, generate summaries, and provide insights based on the document's data.

This kind of portal is particularly useful in scenarios like legal document review, academic research, business reporting, and any other context where interacting with large volumes of text-based information is required.



# Prerequisites

## Confluent Cloud

<<<<<<< Updated upstream
Demo:

[JobportalDemo](https://drive.google.com/file/d/17u96OIifigB1-tLUkJeOt06sZRRhp-gz/view?usp=drive_link)

You need a working account for Confluent Cloud. Sign-up with Confluent Cloud is very easy and you will get a $400 budget for your first trials for free. If you don't have a working Confluent Cloud account please [Sign-up to Confluent Cloud](https://www.confluent.io/confluent-cloud/tryfree/?utm_campaign=tm.campaigns_cd.Q124_EMEA_Stream-Processing-Essentials&utm_source=marketo&utm_medium=workshop).
=======
1. Sign up for a Confluent Cloud account [here](https://www.confluent.io/get-started/).
1. After verifying your email address, access Confluent Cloud sign-in by navigating [here](https://confluent.cloud).
1. When provided with the _username_ and _password_ prompts, fill in your credentials.

   > **Note:** If you're logging in for the first time you will see a wizard that will walk you through the some tutorials. Minimize this as you will walk through these steps in this guide.

1. Create Confluent Cloud API keys by following [this](https://registry.terraform.io/providers/confluentinc/confluent/latest/docs/guides/sample-project#summary) guide.
   > **Note:** This is different than Kafka cluster API keys.

## MongoDB Atlas

1. Sign up for a free MongoDB Atlas account [here](https://www.mongodb.com/).

1. Create an API key pair so Terraform can create resources in the Atlas cluster. Follow the instructions [here](https://registry.terraform.io/providers/mongodb/mongodbatlas/latest/docs#configure-atlas-programmatic-access).

# Setup

1. Clone and enter this repository.

   ```bash
   git clone https://github.com/gopi0518/jobportal-genai.git
   cd jobportal-genai
   ```

1. Create an `.accounts` file by running the following command.

   ```bash
   echo "CONFLUENT_CLOUD_EMAIL=add_your_email\nCONFLUENT_CLOUD_PASSWORD=add_your_password\nexport TF_VAR_confluent_cloud_api_key=\"add_your_api_key\"\nexport TF_VAR_confluent_cloud_api_secret=\"add_your_api_secret\"\nexport TF_VAR_mongodbatlas_public_key=\"add_your_public_key\"\nexport TF_VAR_mongodbatlas_private_key=\"add_your_private_key\"\nexport TF_VAR_mongodbatlas_org_id=\"add_your_org_id\"" > .accounts

   ```

   > **Note:** This repo ignores `.accounts` file

1. Update the `.accounts` file for the following variables with your credentials.

   ```bash
   CONFLUENT_CLOUD_EMAIL=<replace>
   CONFLUENT_CLOUD_PASSWORD=<replace>
   export TF_VAR_confluent_cloud_api_key="<replace>"
   export TF_VAR_confluent_cloud_api_secret="<replace>"
   export TF_VAR_mongodbatlas_public_key="<replace>"
   export TF_VAR_mongodbatlas_private_key="<replace>"
   export TF_VAR_mongodbatlas_org_id="<replace>"
   ```

1. Navigate to the home directory of the project and run `create_env.sh` script. This bash script copies the content of `.accounts` file into a new file called `.env` and append additional variables to it.

   ```bash
   cd jobportal-genai
   ./create_env.sh
   ```

1. Source `.env` file.

   ```bash
   source .env
   ```

   > **Note:** if you don't source `.env` file you'll be prompted to manually provide the values through command line when running Terraform commands.

## Tools
* install git to clone the source
  https://git-scm.com/book/it/v2/Per-Iniziare-Installing-Git
  ```
  yum install git
  ```
* install npm to install UI dependency packages (below example to install npm from yum package)
  ```
  yum install npm
  ```
* install python3
  ```
  yum install python3
  yum install --assumeyes python3-pip
  ```
* install kafka or Confluent Platform (we need the tools kafka-avro-consumer-console and kafka-avro-producer-console), [Downlod Confluent Platform](https://www.confluent.io/get-started/?product=self-managed)
* Local install Confluent CLI, [install the cli](https://docs.confluent.io/confluent-cli/current/install.html) 
* install terraform on your desktop. [Follow the installation guide](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
* I am running still completely on bash on my Intel Mac. Environment Variables are set in `~/.bash_profile`, if you running other OS or use other Shells like zsh, please setup that script can execute kafka tools without absolution path.
* install Python3 on MacOS: [Downland](https://www.python.org/downloads/macos/) and follow the instructions
* Install all the python modules we need;
```bash
pip3 install confluent_kafka
pip3 install PyPDF2
pip3 install pymongo
pip3 install requests
pip3 install fastavro
pip3 install avro
pip3 install jproperties
pip3 install langchain
pip3 install openai
pip3 install langchain_openai
pip3 install google-search-results
pip3 install Flask
pip install --quiet langchain_experimental
```

## API Keys from Confluent Cloud Cluster and Salesforce

For Confluent Cloud: Create API Key in Confluent Cloud via CLI (In my case as OrgAdmin):
```bash
    confluent login
    confluent api-key create --resource cloud --description "API for terraform"
    # It may take a couple of minutes for the API key to be ready.
    # Save the API key and secret. The secret is not retrievable later.
    #+------------+------------------------------------------------------------------+
    #| API Key    | <your generated key>                                             |
    #| API Secret | <your generated secret>                                          |
    #+------------+------------------------------------------------------------------+
```
## Setup
1. Clone and enter this repository.
```
git clone https://github.com/gopi0518/jobportal-genai.git
cd jobportal-genai
```
2. Create an .accounts file by running the following command.
```
echo "CONFLUENT_CLOUD_EMAIL=add_your_email\nCONFLUENT_CLOUD_PASSWORD=add_your_password\nexport TF_VAR_confluent_cloud_api_key=\"add_your_api_key\"\nexport TF_VAR_confluent_cloud_api_secret=\"add_your_api_secret\"\nexport TF_VAR_mongodbatlas_public_key=\"add_your_public_key\"\nexport TF_VAR_mongodbatlas_private_key=\"add_your_private_key\"\nexport TF_VAR_mongodbatlas_org_id=\"add_your_org_id\"" > .accounts
```
3. Update the .accounts file for the following variables with your credentials.
```
CONFLUENT_CLOUD_EMAIL=<replace>
CONFLUENT_CLOUD_PASSWORD=<replace>
export TF_VAR_confluent_cloud_api_key="<replace>"
export TF_VAR_confluent_cloud_api_secret="<replace>"
export TF_VAR_mongodbatlas_public_key="<replace>"
export TF_VAR_mongodbatlas_private_key="<replace>"
export TF_VAR_mongodbatlas_org_id="<replace>"
```
4. Navigate to the home directory of the project and run create_env.sh script. This bash script copies the content of .accounts file into a new file called .env and append additional variables to it.
```
cd jobportal-genai
./create_env.sh
```
5. Source .env file.
```
source .env
```
## Build your cloud infrastructure
1. Navigate to the repo's terraform directory.
```
cd terraform
```
2. Initialize Terraform within the directory.
```
terraform init
```
3. Create the Terraform plan.
```
terraform plan
```
4. Apply the plan to create the infrastructure.
```
terraform apply
```
5. Write the output of terraform to a JSON file. The setup.sh script will parse the JSON file to update the .env file.
```
terraform output -json > ../resources.json
```
6. Run the setup.sh script.
```
cd jobportal-genai
./setup.sh
```
This script achieves the following:

* Creates an API key pair that will be used in connectors' configuration files for authentication purposes.
* Creates an API key pair for Schema Registry
* Creates Tags and business metadata
* Updates the .env file to replace the remaining variables with the newly generated values.
Source .env file
```
source .env file.
```
## Generative AI API we use

We use langchain LLM version 0.1 [Langchain Docu](https://python.langchain.com/docs/get_started/introduction)

HINT:
<table><tr><td>Now, it will cost money. Unfortunately the APIs are not for free. I spend 10$ for open AI, 10$ for ProxyCurl API and SERP API is still in free status.</td></tr></table>

First we need a key which allow us to use OpenAI. Follow steps from [here](https://platform.openai.com/docs/quickstart/account-setup) to create an Account and then an API Key only.

Next Task: Create proxycurl api key. ProxyCurl will be used to scrape Linkedin. Sign Up to proxyurl and buy credits for 10$ (or whatever you think is enough, maybe you start more less) , [follow these Steps](https://nubela.co/proxycurl). You get 10 free credit. Which could be enough for a simple demo.

To be able to search in Google the correct linkedin Profile URL, we need a API key of SERP API from [here](https://serpapi.com/).

Now, put all Keys into `env-vars` file by executing the command:
```
export OPENAI_API_KEY=YOUR openAI Key
export PROXYCURL_API_KEY=YOUR ProxyURL Key
export SERPAPI_API_KEY=Your SRP API KEy
export TAVILY_API_KEY=Your TAVILY KEY
```

Congratulation the preparation is done. This was a huge setup, I know. But all the rest is "one command execution"

## Run the job portal UI (from repo home directory)
```
cd portalUI
npm install --save axios
npm start
```
## Run python code (from repo home directory

```
cd services
python3 jobportalUIService.py -f client.properties -resumereq jobseekerv6 -jobpostreq jobpostreq
python3 jobportalconsumer.py -f client.properties -resumereq jobseekerv6 -jobpostreq jobpostreq -resumeres jobseekerresv1 -jobpostres jobpostresv2
```

# Destroy the Confluent Cloud Resources
If you are finished, you can stop the programs in Terminal with CTRL+c and destroy everything in Confluent Cloud by :
```bash
terraform destroy
``` 

If you will get an error, execute destroy again, till everything is green.
```bash
terraform destroy
``` 

# Licenses
You need a Confluent Cloud Account (new ones get 400$ credit for free).
You need an OPenAI Account, with current credit.
You need a ProxyCurl API Account, with current credits.
Your need SERP API Account, here you will a starting amount of connections. This was enough in my case.

In  Total I spend 20 $ (Open AI, ProxyCurl) and I am still not out of credits.

 
