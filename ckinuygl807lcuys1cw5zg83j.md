## AWS Copilot - Addons, Additional Resources & Monitor

Hello,  👋 In this final post of the AWS Copilot series, we will see how to add addons like storage and additional resources, if any, and monitor the logs of the application.

If you want to check the previous posts of this series, please check them out here.

%[https://hashnode.com/series/aws-copilot-ecs-ckhskugtp07r2s6s14k1l6s56]


## Addons - Storage (DynamoDB)

We've created the demo application running in the container, and we've deployed the container to run in ECS Fargate using AWS Copilot CLI. If we need to add storage like S3 or DynamoDB so that our services can store state, AWS Copilot provides the storage options as add-ons.

Run the below command under coffee service.

```
copilot storage init
```

It creates a new storage resource attached to one of your services, accessible from inside your service container via a friendly environment variable. You can specify either S3 or DynamoDB as the resource type.

For this demo, let's choose DynamoDB as the resource type and add the table name `Coffee` with `CoffeeId` as the hash key. After running this command, the CLI creates an addons subdirectory inside your copilot/service directory if it does not exist. 

```YAML
Parameters:
  App:
    Type: String
    Description: Your application's name.
  Env:
    Type: String
    Description: The environment name your service, job, or workflow is being deployed to.
  Name:
    Type: String
    Description: The name of the service, job, or workflow being deployed.
Resources:
  Coffee:
    Type: AWS::DynamoDB::Table
    DeletionPolicy: Retain
    Properties:
      TableName: !Sub ${App}-${Env}-${Name}-Coffee
      AttributeDefinitions:
        - AttributeName: CoffeeId
          AttributeType: "S"
      BillingMode: PAY_PER_REQUEST
      KeySchema:
        - AttributeName: CoffeeId
          KeyType: HASH

  CoffeeAccessPolicy:
    Type: AWS::IAM::ManagedPolicy
    Properties:
      Description: !Sub
        - Grants CRUD access to the Dynamo DB table ${Table}
        - { Table: !Ref Coffee }
      PolicyDocument:
        Version: 2012-10-17
        Statement:
          - Sid: DDBActions
            Effect: Allow
            Action:
              - dynamodb:BatchGet*
              - dynamodb:DescribeStream
              - dynamodb:DescribeTable
              - dynamodb:Get*
              - dynamodb:Query
              - dynamodb:Scan
              - dynamodb:BatchWrite*
              - dynamodb:Create*
              - dynamodb:Delete*
              - dynamodb:Update*
              - dynamodb:PutItem
            Resource: !Sub ${ Coffee.Arn}
          - Sid: DDBLSIActions
            Action:
              - dynamodb:Query
              - dynamodb:Scan
            Effect: Allow
            Resource: !Sub ${ Coffee.Arn}/Index/*

Outputs:
  CoffeeName:
    Description: "The name of this DynamoDB."
    Value: !Ref Coffee
  CoffeeAccessPolicy:
    Description: "The IAM::ManagedPolicy to attach to the task role."
    Value: !Ref CoffeeAccessPolicy
```

If you want to deploy the newly created storage add-ons, run `copilot svc deploy` of the service you've added the addons. The DynamoDB Table `Coffee` will be created.

### .NET Core Amazon DynamoDB Client

Configure the Amazon DynamoDB .NET Client to use the Coffee Table Name. As you've seen, the table name is `Coffee` prefixed with the `ApplicationName-EnvironmentName-ServiceName.` These values are exposed in the environment variable to the ECS containers.

```csharp
     var config = new DynamoDBContextConfig  { 
        TableNamePrefix = $"{Environment.GetEnvironmentVariable("COPILOT_APPLICATION_NAME")}" + 
        $"-{Environment.GetEnvironmentVariable("COPILOT_ENVIRONMENT_NAME")}" + 
        $"-{Environment.GetEnvironmentVariable("COPILOT_SERVICE_NAME")}-"
    };
    services.AddAWSService<IAmazonDynamoDB>();
    services.AddTransient<DynamoDBContext>(c => new DynamoDBContext(c.GetService<IAmazonDynamoDB>(), config));
```

Add the necessary annotations for using the model with DynamoDB Client.

```
public class Coffee
{
    [DynamoDBHashKey]
    public string CoffeeId { get; set; }
    public string CoffeeName { get; set; }
}
```

You can do the CRUD operations on the DynamoDB model created by AWS Copilot addons.

```csharp
    public  async Task<List<Coffee>> GetAll() {
        var result = this._context.ScanAsync<Coffee>(new List<ScanCondition>());
        return await result.GetRemainingAsync();
    }

    public  async Task<Coffee> GetById(string coffeeId) {
       return await this._context.LoadAsync<Coffee>(coffeeId);        
    }

    public async Task<Coffee> Create(Coffee coffee) {
        await _context.SaveAsync(coffee);
        return await _context.LoadAsync<Coffee>(coffee.CoffeeId);
    }
```

Access the CoffeeService Endpoints that do the CRUD operations on the DynamoDB models.

## Additional Resources

You can add your own custom CloudFormation templates in the addons directory of the services. 

An addon template can be any valid CloudFormation template. However, by default, Copilot will pass the App, Env, and Name Parameters; you can customize your resource properties with Conditions or Mappings.

If you need to access your Resources from your ECS task, make sure to:

1. Define an IAM ManagedPolicy resource in your template that holds the permissions for your task and add an Output so that the permission is injected into your ECS Task Role.
2. Create an Output for any value you want to be injected as an environment variable to your ECS tasks.

E.g., For this, we will create the EventBridge - EventBus Cloudformation template as an additional resource.

```
  OrderEventBus:
    Type: 'AWS::Events::EventBus'
    Properties: 
      Name: !Sub ${App}-${Env}-OrderEventBus

  OrderEventBusAccessPolicy:
    Type: AWS::IAM::ManagedPolicy
    Properties:
      Description: !Sub
        - Grants Push access to the EventBus ${OrderEventBus}
        - { EventBus: !Ref OrderEventBus }
      PolicyDocument:
        Version: 2012-10-17
        Statement:
          - Sid: EventBusActions
            Effect: Allow
            Action:
              - events:PutEvents
            Resource: !Sub ${ OrderEventBus.Arn}
```

Output:

```
Outputs:
  OrderEventBus:
    Description: "The name of the EventBusName."
    Value: !Ref OrderEventBus
```

## Monitor the Service

When you deploy the service, you've to monitor the service logs to observe the application errors and messages to understand what's going on. Copilot has the option to monitor the service logs.

Run the below command to display the logs of a deployed service.

``` bash
copilot svc logs --follow
```

## Conclusion

In this series, we see AWS Copilot powers up the building's entire infrastructure and workflow, pushing and launching your container on AWS. You can build an entire application infrastructure with microservices, load balancer, container registries, storage add-ons, and additional resources.

The source code of this demo is [here](https://github.com/ksivamuthu/copilot-ecs-dotnet-core-demo/). Follow the repo for more updates on this demo.

If you like this post, please do follow me, like, and comment. Your suggestions are welcome. And if you have any questions, please feel free to ask or reach me at my Twitter [ksivamuthu](https://twitter.com/ksivamuthu).

> Follow the light of life. You will never walk in the darkness.