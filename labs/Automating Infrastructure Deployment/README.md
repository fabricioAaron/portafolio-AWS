# Challenge Lab: Automating Infrastructure Deployment

## Scenario

Up to this point, the café staff members have created their Amazon Web Services (AWS) resources and manually configured their applications mostly by using the AWS Management Console. This approach worked well as a way for the café to get started with a web presence quickly. However, the staff members are finding it challenging to replicate their deployments to new AWS Regions so that they can support new café locations in multiple countries. They would also like to have separate development and production environments that reliably have matching configurations.

In this challenge lab, you take on the role of Sofía as you work to automate the café's deployments and replicate them to another AWS Region.

## Lab overview and objectives

In this lab, you gain experience with creating AWS CloudFormation templates. You use the templates to create and update CloudFormation stacks. The stacks create and manage updates to resources in multiple AWS service areas in your AWS account. You practice by using AWS CodeCommit to control the version of your templates. You also observe how you can use AWS CodePipeline to automate stack updates.

After completing this lab, you should be able to do the following:

- Deploy a virtual private cloud (VPC) networking layer by using a CloudFormation template.
- Deploy an application layer by using a CloudFormation template.
- Use Git to invoke CodePipeline and to create or update stacks from templates that are stored in CodeCommit.
- Duplicate network and application resources to another AWS Region by using CloudFormation.

When you start the lab, the following resources are already created for you in the AWS account:

In this challenge lab, you will encounter a few tasks where step-by-step instructions are not provided. You must figure out how to complete the tasks on your own.

## Duration

This lab requires approximately 90 minutes to complete.

## AWS service restrictions

In this lab environment, access to AWS services and service actions might be restricted to the ones that are needed to complete the lab instructions. You might encounter errors if you attempt to access other services or perform actions beyond the ones that are described in this lab.

## Accessing the AWS Management Console

1. At the top of these instructions, choose **Start Lab**.
2. The lab session starts.
3. A timer displays at the top of the page and shows the time remaining in the session.  
   - Tip: If you need more time to complete the lab, choose **Start Lab** again to restart the timer for the environment.
4. Before you continue, wait until the circle icon to the right of **AWS** in the upper-left corner turns green.
5. At the top of these instructions, choose the green circle next to **AWS**.  
   - This option opens the AWS Management Console in a new browser tab. The system automatically signs you in.  
   - Tip: If a new browser tab does not open, a banner or icon at the top of your browser will indicate that your browser is preventing the site from opening pop-up windows. Choose the banner or icon, and choose **Allow pop-ups**.
6. Arrange the AWS Management Console tab so that it displays alongside these instructions. Ideally, you should be able to see both browser tabs at the same time so that you can follow the lab steps.

## A business request: Creating a static website for the café by using CloudFormation (challenge 1)

The café would like to start using CloudFormation to create and maintain resources in the AWS account. As a first attempt at this process, you take on the role of Sofía and create a CloudFormation template that can be used to create an Amazon Simple Storage Service (Amazon S3) bucket. Then, you add more detail to the template so that when you update the stack, it configures the bucket to host a static website for the café.

## Task 1: Connecting to the IDE on the EC2 instance

In this first task, you will connect to VS Code IDE and configure the environment to support the development that you will work on during the rest of the lab.

1. At the top of these instructions, choose **i AWS Details**.
2. Copy values from the table for the following and paste it into an editor of your choice for use later.
   - `LabIDEURL`
   - `LabIDEPassword`
3. In a new browser tab, paste the value for `LabIDEURL` to open the VS Code IDE.
4. On the prompt window **Welcome to code-server**:
   - Enter the value for `LabIDEPassword` you copied to the editor earlier.
   - Choose **Submit** to open the VS Code IDE similar to below.  
     - Note: User Interface is displayed as shown below.
5. The IDE includes the following:
   - A bash terminal in the bottom-right panel.
   - A file browser in the left panel that shows files in the `/home/ec2-user/environment` directory on the instance.
   - A file editor in the upper-right panel. If you select a file in the file browser, such as the `README.md` file, it displays in the editor.

## Task 2: Creating a CloudFormation template from scratch

In this task, you create a CloudFormation template that creates an S3 bucket. You then run an AWS Command Line Interface (AWS CLI) command that created the CloudFormation stack. (The stack is the resource that creates the bucket.)

In the VS Code IDE, choose **Menu**, choose **File** and choose **New File**. Save the new file as `S3.yaml`.

At the top of the file, add the following two lines:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description:
```

Next, add the following three lines to your file:

```yaml
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
```


Tip: Make sure that you keep the correct number of spaces for each indentation level. The Resources line should have no indentation. The S3Bucket line should be indented by two spaces. Finally, the Type: AWS::S3::Bucket line should be indented by four spaces.

CloudFormation supports the YAML version 1.1 specification with a few exceptions. For more information about YAML, see the YAML website.

11. Add a description, such as "cafe S3 template", on the Description line. Before you start your description, be sure that you have a space after the colon (:).

12. After you enter the description, save the changes to the file.

In the guided lab earlier in this module, you used the AWS Management Console to create a CloudFormation stack. Here, you use the AWS CLI instead.

13. In the VS Code IDE Bash terminal, run the following two lines of code:


```bash
aws configure get region
aws cloudformation create-stack --stack-name CreateBucket --template-body file://S3.yaml

```

The first line of code returns the default AWS Region of the AWS CLI client that is installed on the IDE. You can modify the default AWS Region by running the `aws configure` command. However, for this lab, you should leave the default Region.

The second line of code creates a stack that uses the template you defined. Because you did not specify the Region in the command, the stack is created in the default Region.

If the `create-stack` command ran successfully, you should see some output that is formatted in JSON. This output should indicate a `StackId`.

The following diagram illustrates the actions that you just completed.

14 . In the AWS Management Console, navigate to the CloudFormation console, and observe the details of the `CreateBucket` stack.

For example, look at the information in the **Events**, **Resources**, **Outputs**, and **Template** tabs.

15. Navigate to the Amazon S3 console to observe the bucket that your template created.

Tip: The bucket has the bucket name `createbucket-s3bucket-<random-string>`.

### Answering questions about the CloudFormation stack

Your answers are recorded when you choose Submit at the end of the lab.

16. To access the questions in this lab, at the top of these instructions, choose **AWS Details**.

17. Choose the **Access the multiple choice questions** link.

18. In the page that you loaded, submit answers to the following questions:

- Question 1: Was an S3 bucket created, even if you did not specify a name for the bucket? If so, what name was it given?
- Question 2: What Region was the bucket created in, and why was it created in this Region?
- Question 3: To define an S3 bucket, how many lines of code did you need to enter in the `Resources:` section of the template file?

Note: Leave the browser tab with the questions open so that you can return to it later in the lab.


## Task 3: Configuring the bucket as a website and updating the stack

In this next task, you update the CloudFormation template. The update configures the S3 bucket to host a static website. This task is similar to the results from the module 3 challenge lab. In that challenge lab, you created and configured the S3 bucket manually by using the AWS Management Console. However, in this lab, you configure the bucket by using a CloudFormation template.

Next, you set bucket ownership controls and public access, and then upload the static website assets to the bucket.

To complete this task, run the following commands in the IDE Bash terminal (replace all three occurrences of `<BUCKET-NAME>` with your actual bucket name):

```bash
#1. Download the website files
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/15-lab-mod11-challenge-CFn/s3/static-website.zip
unzip static-website.zip -d static
cd static
#2. Set the ownership controls on the bucket
aws s3api put-bucket-ownership-controls --bucket <BUCKET-NAME> --ownership-controls Rules=[{ObjectOwnership=BucketOwnerPreferred}]
#3. Set the public access block settings on the bucket
aws s3api put-public-access-block --bucket <BUCKET-NAME> --public-access-block-configuration "BlockPublicAcls=false,RestrictPublicBuckets=false,IgnorePublicAcls=false,BlockPublicPolicy=false"
#4. Copy the website files to the bucket
aws s3 cp --recursive . s3://<BUCKET-NAME>/ --acl public-read
```



