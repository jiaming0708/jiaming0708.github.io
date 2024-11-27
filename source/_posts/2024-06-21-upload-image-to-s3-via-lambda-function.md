---
title: '[DotnetCore] Lambda Function 實作上傳圖片至 S3'
date: 2024-06-21 13:56:09
updated: 2024-06-21 13:56:09
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
- AWS
thumbnail:
---

透過 AWS Lambda Function 來做為中介上傳檔案到 S3，底下會分享兩種不同的 body 內容如何用 c# 實作。

<!-- more -->

## 建立 Lambda function

先安裝 dotnet AWS 的工具及範本

```shell
dotnet tool install -g Amazon.Lambda.Tools
dotnet new -i "Amazon.Lambda.Templates::*"
```

範本有以下這些

```
Template Name       Short Name                                    Language  Tags
------------------  --------------------------------------------  --------  --------------------------------
AWS Message Pro...  serverless.Messaging                          [C#]      AWS/Lambda/Serverless
Empty Top-level...  lambda.EmptyTopLevelFunction                  [C#]      AWS/Lambda/Serverless
Lambda Annotati...  serverless.Annotations                        [C#]      AWS/Lambda/Serverless
Lambda ASP.NET ...  serverless.AspNetCoreMinimalAPI               [C#]      AWS/Lambda/Serverless
Lambda ASP.NET ...  serverless.AspNetCoreWebAPI                   [C#],F#   AWS/Lambda/Serverless
Lambda ASP.NET ...  serverless.image.AspNetCoreWebAPI             [C#],F#   AWS/Lambda/Serverless
Lambda ASP.NET ...  serverless.AspNetCoreWebApp                   [C#]      AWS/Lambda/Serverless
Lambda Custom R...  lambda.CustomRuntimeFunction                  [C#],F#   AWS/Lambda/Function
Lambda Detect I...  lambda.DetectImageLabels                      [C#],F#   AWS/Lambda/Function
Lambda Empty Fu...  lambda.EmptyFunction                          [C#],F#   AWS/Lambda/Function
Lambda Empty Fu...  lambda.image.EmptyFunction                    [C#],F#   AWS/Lambda/Function
Lambda Empty Se...  serverless.EmptyServerless                    [C#],F#   AWS/Lambda/Serverless
Lambda Empty Se...  serverless.image.EmptyServerless              [C#],F#   AWS/Lambda/Serverless
Lambda Function...  lambda.NativeAOT                              [C#],F#   AWS/Lambda/Function
Lambda Function...  lambda.Powertools                             [C#]      AWS/Lambda/Function/Powertools
Lambda Giraffe ...  serverless.Giraffe                            F#        AWS/Lambda/Serverless
Lambda Serverle...  serverless.Powertools                         [C#]      AWS/Lambda/Serverless/Powertools
Lambda Simple A...  lambda.SimpleApplicationLoadBalancerFunction  [C#]      AWS/Lambda/Function
Lambda Simple D...  lambda.DynamoDB                               [C#],F#   AWS/Lambda/Function
Lambda Simple K...  lambda.KinesisFirehose                        [C#]      AWS/Lambda/Function
Lambda Simple K...  lambda.Kinesis                                [C#],F#   AWS/Lambda/Function
Lambda Simple S...  lambda.S3                                     [C#],F#   AWS/Lambda/Function
Lambda Simple S...  lambda.SNS                                    [C#]      AWS/Lambda/Function
Lambda Simple S...  lambda.SQS                                    [C#]      AWS/Lambda/Function
Lex Book Trip S...  lambda.LexBookTripSample                      [C#]      AWS/Lambda/Function
Order Flowers C...  lambda.OrderFlowersChatbot                    [C#]      AWS/Lambda/Function
Serverless Dete...  serverless.DetectImageLabels                  [C#],F#   AWS/Lambda/Serverless
Serverless proj...  serverless.NativeAOT                          [C#],F#   AWS/Lambda/Serverless
Serverless Simp...  serverless.S3                                 [C#],F#   AWS/Lambda/Serverless
Serverless WebS...  serverless.WebSocketAPI                       [C#]      AWS/Lambda/Serverless
Step Functions ...  serverless.StepFunctionsHelloWorld            [C#],F#   AWS/Lambda/Serverless
```

選擇 `lambda.EmptyFunction` 來實作

```shell
dotnet new lambda.EmptyFunction --name UploadImage --profile default --region us-east-2
```

產生後會看到 `UploadImage` 的目錄內有 src 及 test 的子目錄，程式的入口點在 `src/Function.cs`，內容如下

```c#
using Amazon.Lambda.Core;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace UploadImage;

public class Function
{
    
    /// <summary>
    /// A simple function that takes a string and does a ToUpper
    /// </summary>
    /// <param name="input">The event for the Lambda function handler to process.</param>
    /// <param name="context">The ILambdaContext that provides methods for logging and describing the Lambda environment.</param>
    /// <returns></returns>
    public string FunctionHandler(string input, ILambdaContext context)
    {
        return input.ToUpper();
    }
}
```

## Base64

使用 `APIGatewayProxyRequest` 來取得請求的資料，並且使用 Base64 轉成 byte 再使用 S3 的 `TransferUtilityUploadRequest` 物件上傳

```c#
public class Function
{
    private static readonly string bucketName = "your-s3-bucket-name";
    private static readonly IAmazonS3 s3Client = new AmazonS3Client();

    public async Task<APIGatewayProxyResponse> FunctionHandler(APIGatewayProxyRequest input, ILambdaContext context)
    {
        try
        {
            var imageBytes = Convert.FromBase64String(input.Body);
            var key = Guid.NewGuid().ToString() + ".jpg";

            using (var stream = new MemoryStream(imageBytes))
            {
                var uploadRequest = new TransferUtilityUploadRequest
                {
                    InputStream = stream,
                    Key = key,
                    BucketName = bucketName,
                    ContentType = "image/jpeg"
                };

                var fileTransferUtility = new TransferUtility(s3Client);
                await fileTransferUtility.UploadAsync(uploadRequest);
            }

            return new APIGatewayProxyResponse
            {
                StatusCode = 200,
                Body = $"Image uploaded successfully with key: {key}",
                Headers = new Dictionary<string, string> { { "Content-Type", "text/plain" } }
            };
        }
        catch (Exception ex)
        {
            return new APIGatewayProxyResponse
            {
                StatusCode = 500,
                Body = $"Error: {ex.Message}",
                Headers = new Dictionary<string, string> { { "Content-Type", "text/plain" } }
            };
        }
    }
}
```

## multipart/form-data

安裝 [HttpMultipartParser](https://github.com/Http-Multipart-Data-Parser/Http-Multipart-Data-Parser) 套件來做解析，可以輕鬆地處理掉自己寫 parser 的麻煩

```c#
    public class Function
    {
        private static readonly string BucketName = "your-s3-bucket-name";
        private static readonly IAmazonS3 S3Client = new AmazonS3Client();

        public APIGatewayProxyResponse FunctionHandler(APIGatewayProxyRequest request, ILambdaContext context)
        {
            try
            {
                var stream = new MemoryStream(Convert.FromBase64String(request.Body));
                var multipart = MultipartFormDataParser.ParseAsync(stream);

                foreach (var file in multipart.Files)
                {
                    using (var stream = new MemoryStream(file.Content))
                    {
                        var uploadRequest = new TransferUtilityUploadRequest
                        {
                            InputStream = stream,
                            Key = file.FileName,
                            BucketName = BucketName,
                            ContentType = file.ContentType
                        };

                        var transferUtility = new TransferUtility(S3Client);
                        transferUtility.Upload(uploadRequest);
                    }
                }

                return new APIGatewayProxyResponse
                {
                    StatusCode = (int)HttpStatusCode.OK,
                    Body = "File(s) uploaded successfully"
                };
            }
            catch (Exception ex)
            {
                context.Logger.LogLine($"Error: {ex.Message}");
                return new APIGatewayProxyResponse
                {
                    StatusCode = (int)HttpStatusCode.InternalServerError,
                    Body = $"Error: {ex.Message}"
                };
            }
        }
    }
```
