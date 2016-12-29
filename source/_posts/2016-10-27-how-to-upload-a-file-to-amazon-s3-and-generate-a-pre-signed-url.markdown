---
layout: post
title: "How to upload a file to Amazon S3 and generate a pre-signed URL"
date: 2016-10-27 09:32:43 -0500
comments: true
categories: 
- Python
- Amazon S3
Category: Development
---

```python
import datetime
import StringIO
import xlwt

from boto3.session import Session
from boto3 import client

# Create a file, for example, an Excel document.
xls = xlwt.Workbook()
ws = xls.add_sheet('A Test Sheet')
ws.write(0, 0, 1234.56)

file_name = 'example.xls'
output = StringIO.StringIO()
xls.save(output)

# Save the file to S3
session = Session(
    aws_access_key_id=config['AWS_ACCESS_KEY_ID'],
    aws_secret_access_key=config['AWS_SECRET_ACCESS_KEY'],
)
s3 = session.resource('s3')
s3_bucket_name = 'YourBucket'
s3_file_key = 'dir1/dir2/{}'.format(file_name)

result = s3.Bucket(s3_bucket_name).put_object(
    Key=s3_file_key,
    Body=output.getvalue()
)

# Generate a pre-signed URL
s3_client = client(
    's3',
    region_name='us-east-1',
    aws_access_key_id=config['AWS_ACCESS_KEY_ID'],
    aws_secret_access_key=config['AWS_SECRET_ACCESS_KEY'],
)

url = s3_client.generate_presigned_url(
    ClientMethod='get_object',
    Params={
        'Bucket': s3_bucket_name,
        'Key': s3_file_key,
    },
    ExpiresIn=60 * 60 * 24,  # Default: 3600
)

print(url)
# https://s3.amazonaws.com/YourBucket/dir1/dir2/example.xls?AWSAccessKeyId=ATPAJNUDN3ENT2I4S6EF&Expires=1477582080&Signature=POvreXScdYNES98SPFeAN3y12DL%3D
```