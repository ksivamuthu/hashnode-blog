## Amazon Kendra - Enterprise Search powered by ML

We are always asking questions to expand our knowledge. We need quick and relevant answers to our questions in everyday life. What's the time now? How is the weather today? What's the travel time to work? How many cups of sugar in this recipe? When will this pandemic end? And we have smart search engines to ask.

Machine learning is continuously making search engines smarter. And we are expecting the speedy, accurate and personalized results. The search interface is available on traditional platforms, websites, and modern conversational platforms such as chatbots and voice assistant devices.

Great so far. But let's talk about enterprise search use cases. The search seems a lot more troublesome when it comes to enterprise. The data from the various data sources and legacy applications hamper internal data searches.

The promising Amazon Kendra is a highly accurate and easy to use enterprise search that's powered by machine learning. Kendra delivers powerful natural language search capabilities to your websites and applications so your end users can easily find the information they need within the vast amount of content spread across your enterprise.

Are you ready to see the benefits of AWS Kendra in action? Let's start.

## Prerequisites

1. AWS Account
2. Sample Documents / FAQs in one of the supported data sources (S3, Onedrive, Sharepoint Online, Salesforce, ServiceNow, RDS)
3. ☕️ Coffee to sip when you are waiting on creating an index.

## Setup Amazon Kendra

### Create Index

Open the Amazon Kendra console at https://console.aws.amazon.com/kendra. Create an index by specifying the index name, description, and role with [required policies](https://docs.aws.amazon.com/kendra/latest/dg/iam-roles.html#iam-roles-index).

![create index](https://cdn.hashnode.com/res/hashnode/image/upload/v1605923104847/gRpbyeqD7.gif)

### Add data sources

Amazon Kendra provides an easy way to add your data sources that it should index.  You can schedule the sync periodically or sync on demand. The sync will update the index of new, updated, and deleted files from your document source repositories.

Supported data sources:

1. Amazon S3
2. RDS
3. Microsoft OneDrive
4. Salesforce
5. ServiceNow
6. Microsoft SharePoint


You can directly add the documents to the index through the document addition API. And also, you can upload the FAQs with answers and an optional link to the document.

In this demo, I've added publicly available documents from Wikipedia and websites to the S3 bucket. Now let's add S3 bucket as the data source to the index. 

Click "Add data sources" in the index. Choose "Amazon S3" from the list of data source connectors and enter the data source location, IAM role, and set sync run schedule. Review the attributes and create the data source. 

![add data source](https://cdn.hashnode.com/res/hashnode/image/upload/v1605923108051/J83J2PaJ7.gif)

### Test and deploy.

You can test and tune the search experience directly in the search console. In the Amazon Kendra console, choose your index and then select Search console.

Enter your search text and click the search icon. Amazon Kendra returns the results of the search.

Amazon Kendra - ML-powered Enterprise search differs from the traditional search by understanding the context of the question and return the most relevant data.  The powerful natural language processing behind Amazon Kendra processes the user's natural language questions or keywords to search the specific answers anywhere in data. 

In this demo, I'm searching for questions such as 

* How long are the ISS missions?
* How fast the space stations travel?
* What is the pH of pure water?
* When is Walt Disney World founded?

Amazon Kendra brought up the specific answer for the question we asked using powerful ML and NLP capabilities and highlighted the response as below. Isn't that cool?

![Alt Text](https://cdn.hashnode.com/res/hashnode/image/upload/v1605923110727/BpOvAqdrp.png)

And another question...

![Alt Text](https://cdn.hashnode.com/res/hashnode/image/upload/v1605923113010/SduZ10UGJ.png)

### Feedback & Retrain

Amazon Kendra powered search results get better as end-users use the service. Kendra actively retrains the deep learning models build for your data set. The usage and feedback patterns improve search accuracy. You can also have the option to fine-tune results by adjusting the importance of specific data sources.

## Production workloads

You pay only for what you use for Amazon Kendra Service. Amazon Kendra comes in two editions.

1. Kendra Enterprise Edition provides a high-availability service for production workloads.
2. Kendra Developer Edition provides developers with a lower- cost option to build a proof-of-concept.  

You can add more capacity - query-based or storage-based in Enterprise Edition based on scale. Please checkout the Amazon Kendra pricing options [here](https://aws.amazon.com/kendra/pricing/).

## Demo

![Demo](https://cdn.hashnode.com/res/hashnode/image/upload/v1605923115692/6rng0lVbD.gif)

## Final Thoughts

Amazon Kendra uses deep learning models to understand natural and complex language queries and document content and structures for a wide range of internal use cases like IT, financial services, insurance, pharmaceuticals, industrial manufacturing, energy, legal, media and entertainment, travel and hospitality, healthcare, HR, news, telecommunications, and automotive. Its benefits are great.

1. Natural Language Understanding
2. Easy to add - connectors to the popular data sources
3. Relevance Tuning
4. Document Ranking
5. Reading comprehension
6. Focusing on the answer - not on links
7. Incremental learning

Do you want to see the demo live? Do you want to know how to integrate the Amazon Kendra search into your enterprise chatbot? Please join, as I'm going to present a live session at my youtube channel / [twitch](https://twitch.tv/ksivamuthu) on June 15, 2020 - 6.30 pm ET discussing Amazon Kendra and demo integrated with the chatbot. 

{% youtube yqABUbUahN0 %}

Please do like, share, and comment. DM me at my twitter @ksivamuthu if you've any questions or to discuss ML/IoT use cases for your business to solve.