# AWS re:Invent 2022 Announcements - Data, Infrastructure & Security Highlights

Amazon made a couple of announcements today (November 29, 2022) at AWS re: Invent in Las Vegas, focusing on Data, Analytics, Infrastructure & Security. We will see the announcements that excite me in this blog post.

### Data & Analytics:

1.  Zero ETL future
    
    When you build a data platform - ETL takes primary effort. AWS committed to building the Zero ETL future and made two great announcements toward that.
    
    *   [Amazon Aurora zero-ETL integration with Amazon Redshift](https://aws.amazon.com/about-aws/whats-new/2022/11/amazon-aurora-zero-etl-integration-redshift/)
        
        *   enables near-real-time analytics and ML on transaction data.
            
        *   consolidate data from multiple Aurora database
            
        *   continuous updates
            
        *   serverless - no infrastructure to manage
            
    *   Amazon Redshift Integration for Apache Spark
        
        *   Now, you can run Spark queries on REshift data from EMR, Glue, and Sagemaker within seconds. No need to move the data
            
2.  [Amazon Datazone](https://aws.amazon.com/datazone/) - ML based Data management service
    
    *   A data management service that helps orgs catalog, share and govern data.
        
    *   Integrates with Redshift, Athena and Quicksights and provides APIs to third party sources
        
    *   AWS Clean Room - Data sharing / masking services.
        
        1.  I think AWS is running out the names for the product. ;-)
            
3.  Amazon QuickSight - ML powered
    
    *   It includes AI-enhanced automated data preparation, making it fast & easy to enable data for natural language questions.
        
4.  [Amazon OpenSearch Serverless](https://aws.amazon.com/blogs/aws/preview-amazon-opensearch-serverless-run-search-and-analytics-workloads-without-managing-clusters/)
    
    *   Run search and analytics workloads without managing clusters.
        
    *   It’s one of the features I expected - finally, it got announced.
        
5.  [Amazon RDS Blue/Green Deployments](https://aws.amazon.com/blogs/aws/new-fully-managed-blue-green-deployments-in-amazon-aurora-and-amazon-rds/)
    
    *   A new feature for Amazon Aurora with MySQL compatibility, Amazon RDS for MySQL, and Amazon RDS for MariaDB that enables you to make database updates safer, simpler, and faster.
        
    *   With just a few steps, you can use Blue/Green Deployments to create a separate, synchronized, fully managed staging environment that mirrors the production environment. The staging environment clones your production environment’s primary database and in-Region read replicas. Blue/Green Deployments keep these two environments in sync using logical replication
        

### Infrastructure

1.  Graviton: NEW 3rd generation processor. 25% faster & use 60% LESS energy.
    
    1.  Strong momentum for Graviton
        
    2.  500x time increase: delivering highest performing #MachineLearning workloads. 70% lower cost per inference.
        
2.  [AWS announces EC2 Inf2 instances](https://aws.amazon.com/about-aws/whats-new/2022/11/aws-announces-amazon-ec2-inf2-instances-preview/)
    
3.  [AWS Simspace Weave](https://aws.amazon.com/blogs/aws/new-aws-simspace-weaver-build-large-scale-spatial-simulations-in-the-cloud/)r - Run large-scale spatial simulations in the cloud
    

### Security

1.  [Amazon Security Lake](https://aws.amazon.com/blogs/aws/preview-amazon-security-lake-a-purpose-built-customer-owned-data-lake-service/): centralizes security data of your organization from AWS and third-party systems for quick risk response. Supports OCSF standard format. Amazon Security Lake is the first service/initiative to support OCSF open standard format - to normalize the structure for security findings across vendors and products.
    
2.  AWS Guard Duty - Runtime vulnerability detection for containers., attempts to access host nodes, etc. This feature will be available soon and integrates with Amazon EKS
    

### Industries

1.  [Amazon Omics](https://aws.amazon.com/blogs/aws/introducing-amazon-omics-a-purpose-built-service-to-store-query-and-analyze-genomic-and-biological-data-at-scale/) - Genome large-scale multimodal analysis that integrates with Amazon Healthlake or Sagemaker
    
2.  [AWS Supply Chain](https://aws.amazon.com/about-aws/whats-new/2022/11/aws-supply-chain-preview/#): improves visibility into systems to lower costs & improve #CX - supply chain resiliency.
    

### Summary

There were exciting announcements in today's keynote focused on Data, Analytics & Infrastructure mostly. I'm excited to about learning these new announcements and help enterprises on their successful journey - that innovates, analyzes, and scales their enterprise data analytics platform.

I'm Siva - working as Principal Architect at [Computer Enterprises Inc](https://www.ceiamerica.com) from Orlando. I'm an AWS Community builder. I will write a lot about Cloud, Containers, IoT, and Devops. If you are interested, please follow me @ksivamuthu on Twitter or check out my blogs at blog.sivamuthukumar.com!