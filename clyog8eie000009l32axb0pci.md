---
title: "Using AWS S3 without hosting your .Net app on AWS"
datePublished: Tue Jul 16 2024 13:28:19 GMT+0000 (Coordinated Universal Time)
cuid: clyog8eie000009l32axb0pci
slug: using-aws-s3-without-hosting-your-net-app-on-aws
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721136370463/76ee0f6a-f6aa-4685-9e91-36f0626cc65d.jpeg
tags: aws, net, aws-s3

---

First, you have to add libraries to your project. [AWSSDK.S3](https://www.nuget.org/packages/AWSSDK.S3) and [AWSSDK.Extensions.NETCore.Setup](https://www.nuget.org/packages/AWSSDK.Extensions.NETCore.Setup).

```bash
dotnet add package AWSSDK.S3
dotnet add AWSSDK.Extensions.NETCore.Setup
```

## Add AwsOptions

To add AWS configuration, we will use an option pattern. You have to create a non-abstract class with the public constructor.

```csharp
public class AwsOptions {
    public const string SectionName = "AWS";
    public string BucketName { get; set; } = string.Empty;
    public string Environment { get; set; } = string.Empty;
    public string AccessKey { get; set; } = string.Empty;
    public string SecretKey { get; set; } = string.Empty;
}
```

Then, go to your appsettings file and add a section for AWS. It's essential to keep the same names of properties.

```json
"AWS": {
    "BucketName": "your_bucket_name",
    "Environment": "your_environment",
    "AccessKey": "<secret>",
    "SecretKey": "<secret>"
  }
```

Finally, go to the Program.cs and add options and services.

```csharp
var options = builder.Configuration.GetSection(AwsOptions.SectionName).Get<AwsOptions>();
ArgumentNullException.ThrowIfNull(options);
var awsOptions = builder.Configuration.GetAWSOptions();
awsOptions.Credentials = new BasicAWSCredentials(options.AccessKey, options.SecretKey);
// Choose your region from AWS
awsOptions.Region = RegionEndpoint.EUNorth1;
builder.Services.AddDefaultAWSOptions(awsOptions);
builder.Services.AddAWSService<IAmazonS3>();
```

## Get an Access and a Secret Key from AWS S3

**User creation**

In AWS go to IAM.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719479745976/d9c3e321-7fc2-4d72-8f81-43207d9a3f6e.png align="center")

Choose "Users" from "Access management".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719479835811/c53aea85-6c8f-4238-bfd4-94e0ab4cf52a.png align="center")

Click "Create user".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719479928513/6152712f-75c7-4266-ae0d-598dd3f40199.png align="center")

Input the name and click "Next".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719479982208/c552aa51-5c44-4b27-881a-a33a59416c8c.png align="center")

Set permissions. Amazon recommends using groups. Click "Create group".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719480218473/d34d2423-944a-498e-9cb2-d4aeebcbc06b.png align="center")

Search for policy for S3 and choose what you need. Name the user group and click "Create user group".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719480394078/426857bc-3afb-400f-9c7a-9eb85245b321.png align="center")

After group creation, choose it from the table and click "Next".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719480572123/2e5938a4-35c0-45c8-bd06-190830e3d3f3.png align="center")

Then you will see the summary page. If everything is okay, click "Create user".

**Get keys**

Click your user name.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719480866916/d4535f6f-13c0-4001-9bad-a587901d22f3.png align="center")

Navigate to "Security credentials".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719480926666/f7f8ffd5-e1f7-4b3b-934c-778397a18b83.png align="center")

Go to "Access keys" and click "Create access key".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719480983869/26ee537b-4195-46d3-8b92-9a9799bcfc2d.png align="center")

Choose a use case. For that case, you should choose "Application running outside AWS".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719481076923/f3180061-b405-49fa-8915-3c774ecf4c56.png align="center")

Setting the description tag is optional. Click "Create access key".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719481141079/7bc74db3-5830-49ec-b5af-0806d51af146.png align="center")

Save your access key and secret access key.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719481321735/da62014f-818d-4397-8a57-e5daf2ef678f.png align="center")

## .Net Secret Manager

To keep your secrets locally, we will use [.Net Secret Manager](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-8.0&tabs=windows#secret-manager). In your project directory use the console:

```bash
dotnet user-secrets init
dotnet user-secrets set "AWS:AccessKey" "YourAccessKey"
dotnet user-secrets set "AWS:SecretKey" "YourSecretKey"
```

## Protip

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Remember to store your Access and Secret Key securely! For example, use Azure KeyVault, AWS Secrets Manager, or just environment variables in your hosting configuration.</div>
</div>

# Remarks

You can find the full code example [here](https://github.com/Katarzyna-Kadziolka/Pathmaker/tree/develop/api/Pathmaker)

If you have any questions you can ask me:  
[contact@chihuahuacoder.com](mailto:contact@chihuahuacoder.com)