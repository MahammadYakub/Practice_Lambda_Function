# Practice_Lambda_Function

This project demonstrates how to use AWS Lambda to convert a text file to an MP3 file. 

This project involves the following steps:

1. **Create an S3 Bucket**: To store the text files and MP3 files.
2. **Create a Lambda Function**: To convert text to MP3 using AWS Polly.
3. **Set Up S3 Trigger**: To trigger the Lambda function when a text file is uploaded.
4. **Test the Setup**: Upload a text file to S3 and check the output MP3 file.

### Project Overview

1. **Create an S3 Bucket**: Set up an S3 bucket to store input text files and output MP3 files.
2. **Create a Lambda Function**: Write a Python function that uses AWS Polly to convert text to MP3.
3. **Set Up S3 Trigger**: Configure S3 to trigger the Lambda function when a new text file is uploaded.
4. **Test the Conversion**: Upload a text file and verify the MP3 output.

### Detailed Steps

#### Step 1: Create an S3 Bucket

1. **Log in to AWS Management Console**
   - Navigate to the [AWS Management Console](https://aws.amazon.com/console/).

2. **Go to S3 Service**
   - Search for "S3" in the search bar and select "S3" from the services list.

3. **Create a New Bucket**
   - Click on "Create bucket".
   - Bucket name: Choose a unique name (e.g., `text-to-mp3-bucket-123`).
   - Region: Select a region (e.g., `us-east-1`).
   - Click "Create bucket" to complete the setup.

#### Step 2: Create a Lambda Function

1. **Go to Lambda Service**
   - Search for "Lambda" in the search bar and select "Lambda" from the services list.

2. **Create a New Lambda Function**
   - Click on "Create function".
   - Choose "Author from scratch".
   - Function name: `TextToMP3Converter`
   - Runtime: Python 3.8 or later (e.g., Python 3.9 or 3.10)
   - Role: Choose "Create a new role with basic Lambda permissions" (this role will allow Lambda to write logs to CloudWatch and access S3 and Polly).

3. **Write the Lambda Function Code**
   - In the Lambda function editor, replace the default code with the following Python code:

     ```python
     import json
     import boto3
     import urllib.parse

     s3_client = boto3.client('s3')
     polly_client = boto3.client('polly')

     def lambda_handler(event, context):
         # Extract bucket name and object key from the event
         bucket_name = event['Records'][0]['s3']['bucket']['name']
         object_key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])

         # Download the text file from S3
         response = s3_client.get_object(Bucket=bucket_name, Key=object_key)
         text = response['Body'].read().decode('utf-8')

         # Use Polly to convert text to speech
         polly_response = polly_client.synthesize_speech(
             Text=text,
             OutputFormat='mp3',
             VoiceId='Joanna'  # You can choose different voices
         )

         # Save the MP3 file to S3
         mp3_key = object_key.replace('.txt', '.mp3')
         s3_client.put_object(
             Bucket=bucket_name,
             Key=mp3_key,
             Body=polly_response['AudioStream'].read(),
             ContentType='audio/mpeg'
         )

         return {
             'statusCode': 200,
             'body': json.dumps(f'MP3 file created: {mp3_key}')
         }
     ```

   - Click "Deploy" to save and deploy your Lambda function.

#### Step 3: Set Up S3 Trigger

1. **Go to S3 Bucket**
   - Navigate back to your S3 bucket in the AWS Management Console.

2. **Configure Lambda Trigger**
   - Select the bucket you created (`text-to-mp3-bucket-123`).
   - Go to the "Properties" tab.
   - Scroll down to the "Event notifications" section and click "Create event notification".
   - Name: `LambdaTrigger`
   - Event types: Select "All object create events" to trigger the Lambda function on file uploads.
   - Destination: Choose "Lambda Function" and select the Lambda function you created (`TextToMP3Converter`).
   - Click "Save changes".

3. **Add Permissions**
   - Lambda needs permission to be triggered by S3 and to access Polly. Go to the Lambda function’s configuration page.
   - Click on "Permissions" and then "Execution role".
   - Add a policy to the role that allows `s3:GetObject`, `s3:PutObject`, and `polly:SynthesizeSpeech` for the specific bucket.

     Here’s a sample policy:

     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": [
                     "s3:GetObject",
                     "s3:PutObject"
                 ],
                 "Resource": [
                     "arn:aws:s3:::text-to-mp3-bucket-123",
                     "arn:aws:s3:::text-to-mp3-bucket-123/*"
                 ]
             },
             {
                 "Effect": "Allow",
                 "Action": "polly:SynthesizeSpeech",
                 "Resource": "*"
             }
         ]
     }
     ```

   - Attach this policy to the Lambda execution role.

#### Step 4: Test the Conversion

1. **Upload a Text File to S3**
   - Go to your S3 bucket (`text-to-mp3-bucket-123`).
   - Click on "Upload" and add a text file (e.g., `sample.txt` with some text content).
   - Click "Upload".

2. **Verify Lambda Execution**
   - Go to the [CloudWatch Logs](https://console.aws.amazon.com/cloudwatch/home) service in the AWS Management Console.
   - Select "Logs" and find the log group for your Lambda function (e.g., `/aws/lambda/TextToMP3Converter`).
   - Check the log streams to see if your Lambda function has executed and logged the output.

3. **Check the MP3 File in S3**
   - Go back to your S3 bucket.
   - Look for the MP3 file with the same name as the uploaded text file but with a `.mp3` extension (e.g., `sample.mp3`).

### Summary

We've created a Lambda function that converts text files to MP3 using AWS Polly. This project demonstrates integrating S3 with Lambda and Polly for text-to-speech conversion. You can enhance this by adding error handling, supporting different text formats, or customizing Polly voices and speech parameters.
