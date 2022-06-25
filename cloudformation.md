# AWS CloudFormation
- Is a declarative way to deploy infrastructures
- No resources manually created, code can be version controlled
- Can schedule turn on and off the entire environment

## How CloudFormation works
- Templates have to be uploaded to S3 then referenced to CloudFormation
- Can't update a template but have to upload a newer version of it
- Stacks are defined by name
- Deleting a stack will delete every single artifact that was created by that stack
  
## Deploy CloudFormation templates
- Manual way: use console to input params
- Automated way: editing templates in YAML files then use AWS CLI to deploy them.

## CF Building Blocks
- **Resources**: AWS resources declared in the template (MANDATORY)
- **Params**: the dynamic inputs for the template
- **Mapping**: the static variables
- **Outputs**: references to what has been created
- **Conditional**: list of conditions to perform resource creation
- **Metadata**

- ***For the exam, need to learn how to read CF templates***

## Create stack
- Go to CF => `Stacks` => `Create stack` => `Create with new resource`
- We can create by uploading a ready template, create a new template with designer or use a sample file
- We can either reference the template file in S3 or upload a template file, this file will also be stored in S3

## CF Resources
- Resources are mandatory
- Specify resources under Resources in template file
- There are reference pages to learn how to declare resources
  
## CF Parameters
- Used to create dynamic templates
- To reference a parameter, we need to use `!Ref`
  ```
    DbSubnet1:
        Type: AWS::EC2::Subnet
        Properties:
            VpcId: !Ref MyVPC
  ```
- `!Ref` can also be used to reference to a defined resource
- AWS offers pseudo parameters, these can be used any time and return fixed values depending on the user

## CF Mappings
- Hardcoded variables in templates
- Handy to differentiate between different environments
  ```
    Mappings:
        Mapping01:
            Key01:
                Name: Value01
            Key02:
                Name: Value02
  ```
- Mappings are used when we know the values in advance, if we don't know then we should use parameters
- To find a value in maps, we can use `Fn::FindInMap` function
- `!FindInMap` [**MapName**, **TopLevelKey**, **SecondLevelKey**]
  ```
    !FindInMap [RegionMap, !Ref AWS::RegionName, 32]
  ```
- `AWS::RegionName` is the pseudo param, returns the current region name, `RegionMap` is the Map name, `32` is the secondary key, its value is the AMI ID of 32 bit machine

## CF Outputs
- Used to output the optional values that we can import to another stacks
- Useful to define network for CF and output the VPC and Subnet ID so that following templates can reuse
- If the outputs of template is referenced in another the template, we can't delete the first template
- To export a value, we use `Export`
  ```
    Outputs:
        StackSSHSecurityGroup:
            Description: Sec group for the company
            Value: !Ref MyCompanyWideSecGroup
            Export:
                Name: SSHSecurityGroup
  ```
- To import value, we use `Fn::ImportValue`
  ```
    Resoureces:
        MySecureInstance:
            Type: AWS::EC2::Instance
            Properties:
                AvailabilityZon: us-east-1a
                ImageId: ami-a4sdfaxv
                InstanceType: t2.micro
                SecurityGroups:
                    - !ImportValue: SSHSecurityGroup
  ```
## CF Conditions
- Used to control the creation of the resources or outputs based on a condition
  ```
  Conditions:
    CreateProdResources: !Equals [!Ref EnvType, prod]
  ```
- Conditions can be used on resources, outputs, etc...
  ```
    Resources:
        MountPoint:
            Type: "AWS::EC2::VolumeAttachment"
            Condition: CreateProdResources
  ```
## Intrinsic functions
### Fn::Ref
- Reference to a parameter or resource
- Shorthand is `!Ref`

### Fn::GetAtt
- To get attributes of a resource
  ```
    Resources:
        EC2Instance:
            Type: "AWS::EC2::Instance"
            Properties:
                ImageId: ami-123123
                InstanceType: t2.micro
        NewVolume
            Type: "AWS::EC2::Volume"
            Condition: CreateProdResources
            Properties:
                Size: 100
                AvailabilityZone:
                    !GetAtt EC2Instance.AvailabilityZone
  ```
### Fn::FindInMap
- Use to get values from maps
### Fn::ImportValue
- To import values exported from other teplates
### Fn::Join
- Join values with a delimiter:
  ```
    !Join [ delimiter, [ comma-delimited list of values ] ]
  ```
- This creates a:b:c
  ```
    !Join [ ":", [a, b, c] ]
  ```
### Fn::Sub
- To subtitute values
- **String** must contain ${VariableName} and will subtite them
  ```
    !Sub
        - String
        - { Var1name: Var1Value, Var2Name: Var2Value }
  ```
  ```
    !Sub String
  ```
### Condition functions
- Fn::And
- Fn::Equals
- Fn::If
- Fn::Not
- Fn::Or

## CF Rollback
- When enable rollback, if there is failure during deployment, CF will rollback/delete resources
- After fixing the failure, we should delete the failed stack and create a new one

## ChangeSet
- When we update a stack, ChangeSet will tell what are the changes, will not tell if the update will fail/succeed
## NestedStack
- Stacks are created as part of other stacks
- Create a nested stack by using `AWS::CloudFormation::Stack`
## CrossStack
- Instead of putting all resources into 1 stack, we create multi stacks with related resources and reuse them

## StackSet
- A stack set is a stack for admin account to manage/delete multiple stacks in multiple regions of multiple accounts

## CF Drift
- A drift is when resources are created auto by CF but someone manually changes the config
- CF Drift is to detect the changes, not all resources are supported
- To detect a drift, select a stack, in `Stack Actions` => `Detect Drift`