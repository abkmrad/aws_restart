# AWS Lambda Word Count Lab

This challenge lab teaches you how to use **AWS Lambda, Amazon S3, and Amazon SNS** to count words in a text file and email the result.  

---

## 🎯 Objectives
By the end of this lab, you will be able to:
- Create a **Lambda function** in Python to count words in a text file.
- Configure an **S3 bucket** to automatically trigger the Lambda function when a `.txt` file is uploaded.
- Use **SNS** to send an email (or optional SMS) with the word count.

---

## 🕒 Duration
~90 minutes

---

## 🛠️ Step 1 – Create an SNS Topic
1. Open **Amazon SNS** → **Topics** → **Create topic**.
2. Type: **Standard**  
   Name: `word-count-results`
3. Create a subscription:
   - **Protocol:** Email  
   - **Endpoint:** Your email address
4. Check your inbox and **Confirm subscription**.

> ✅ Optional: Add a second subscription with **Protocol: SMS** and your phone number.

---

## 📂 Step 2 – Create an S3 Bucket
1. Open **Amazon S3** → **Create bucket**.
2. Name it: `word-count-input-<your-unique-id>` (must be globally unique).
3. Keep defaults (Block Public Access **ON**).
4. Click **Create bucket**.

---

## 🐍 Step 3 – Create the Lambda Function
1. Open **AWS Lambda** → **Create function**.
2. **Name:** `WordCountFunction`  
   **Runtime:** Python 3.12  
   **Execution role:** Use existing role → **LambdaAccessRole**
3. Click **Create function**.

### Add environment variable
- Go to **Configuration → Environment variables → Edit**.  
- Add:
  - **Key:** `TOPIC_ARN`
  - **Value:** *(Paste your SNS topic ARN)*

### Add your code
- Go to the **Code** tab.  
- Replace the contents of `lambda_function.py` with your Python script.

### Replace the contents of `lambda_function.py` with this:

```python
import os
import re
import boto3
import urllib.parse

s3 = boto3.client('s3')
sns = boto3.client('sns')

TOPIC_ARN = os.environ.get('TOPIC_ARN', '')

WORD_PATTERN = re.compile(r"\b[\w'-]+\b", flags=re.UNICODE)

def count_words(text: str) -> int:
    return len(WORD_PATTERN.findall(text))

def lambda_handler(event, context):
    for record in event.get('Records', []):
        bucket = record['s3']['bucket']['name']
        key = urllib.parse.unquote_plus(record['s3']['object']['key'])

        obj = s3.get_object(Bucket=bucket, Key=key)
        text = obj['Body'].read().decode('utf-8', errors='ignore')

        word_count = count_words(text)
        message = f"The word count in the {key} file is {word_count}."
        sns.publish(TopicArn=TOPIC_ARN, Subject="Word Count Result", Message=message)

    return {"status": "ok"}
```
- Click **Deploy**.

---

## 🔔 Step 4 – Add S3 Trigger
1. In your Lambda → **Configuration → Triggers → Add trigger**.
2. Choose **S3** and select your bucket.
3. **Event type:** All object create events → `PUT`
4. Optional filter: **Suffix = `.txt`**
5. Click **Add**.  
   *(Triggers are enabled automatically in the new console.)*

---

## 🧪 Step 5 – Test
1. Upload some `.txt` files into your S3 bucket:
   - `short.txt` → “hello world”
   - `long.txt` → a longer paragraph
   - `empty.txt` → blank file
2. Wait a few seconds.
3. Check your **email inbox**:
   - Subject: `Word Count Result`
   - Body: `The word count in <filename> file is nnn.`

---

## 📊 Step 6 – Verify
- Open **CloudWatch Logs** → Find your Lambda logs.  
- Confirm it ran and printed logs for each file.

---

## 📤 Deliverables
For completion, submit:
1. A forwarded email result from SNS.
2. A screenshot of:
   - Lambda **Code** tab
   - Lambda **Triggers** tab

---

## ✅ Conclusion
You have successfully:
- Built a Lambda function to count words.
- Configured S3 to invoke Lambda on file uploads.
- Sent the result through SNS by email (and optional SMS).
