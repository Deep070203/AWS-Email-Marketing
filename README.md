# AWS-Email-Marketing

This README provides code for AWS Lambda FUnction to send an email using AWS services including SES, and EventBridge. It also includes a policy to grant the necessary permissions to the Lambda function.

## Lambda Function Python Code

The Lambda function is responsible for reading contacts from an S3 bucket, personalizing email templates, and sending emails using SES.

### Python Code

```python
import boto3
import csv

# Initialize the boto3 client for S3 and SES
s3_client = boto3.client('s3')
ses_client = boto3.client('ses')

def lambda_handler(event, context):
    # Specify the S3 bucket name
    bucket_name = 'dj-email-marketing'  # Replace with your bucket name

    try:
        # Retrieve the CSV file containing contacts from S3
        csv_file = s3_client.get_object(Bucket=bucket_name, Key='contacts.csv')
        lines = csv_file['Body'].read().decode('utf-8').splitlines()
        
        # Retrieve the HTML email template from S3
        email_template = s3_client.get_object(Bucket=bucket_name, Key='email_template.html')
        email_html = email_template['Body'].read().decode('utf-8')
        
        # Parse the CSV file
        contacts = csv.DictReader(lines)
        
        for contact in contacts:
            # Replace placeholders in the email template with contact information
            personalized_email = email_html.replace('{{FirstName}}', contact['FirstName'])
            
            # Send the email using SES
            response = ses_client.send_email(
                Source='you@yourdomainname.com',  # Replace with your verified "From" address
                Destination={'ToAddresses': [contact['Email']]},
                Message={
                    'Subject': {'Data': 'Your Weekly AWS NewsLetter!', 'Charset': 'UTF-8'},
                    'Body': {'Html': {'Data': personalized_email, 'Charset': 'UTF-8'}}
                }
            )
            print(f"Email sent to {contact['Email']}: Response {response}")
    except Exception as e:
        print(f"An error occurred: {e}")
```
## Lambda Function Test Event

Use the following JSON as a test event to simulate a scheduled execution of the Lambda function. The function does not use this event data but requires an event to trigger.

```json
{
  "comment": "Generic test event for scheduled Lambda execution. The function does not use this event data.",
  "test": true
}
```

## IAM Policy for SES and S3 Permissions

The Lambda function requires permissions to access S3 and SES. The following IAM policy grants the necessary permissions.

### Policy JSON

Update the ARN to use your S3 bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::ttt-email-marketing/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
        }
    ]
}
```
