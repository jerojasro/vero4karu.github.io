---
layout: post
title: "How to send an email with attachment via Amazon SES in Python"
date: 2016-10-26 14:19:26 -0500
comments: true
categories: Python, Amazon SES
---

```python
import xlwt
from boto3 import client

email_from = 'from@example.com'
email_to = 'to@example.com'

# Create a workbook
xls = xlwt.Workbook()
# Create a sheet
sheet = xls.add_sheet('Sheet name')
# Set styles for headers and dates
header_style = xlwt.easyxf('font: bold True;')
date_format = xlwt.XFStyle()
date_format.num_format_str = 'dd/mm/yyyy hh:mm:ss'

# Header
labels = [
    u'Title 1',
    u'Title 2',
]
content = [labels]

# Get data for other rows
for row in get_rows_with_content():
    content.append(list(row))

# Write it down
for row in range(len(content)):
    for col in range(len(content[row])):
        data = content[row][col]
        # If it's a header
        if row == 0:
            sheet.write(row, col, data, header_style)
            continue
        # If it's a date
        if type(data) == datetime.datetime:
            sheet.write(row, col, data, date_format)
        else:
            sheet.write(row, col, data)

# Set filename
today = datetime.date.today()
file_name = 'my_file_{}.xls'.format(today.strftime("%Y-%m-%d"))

# Save the file
output = StringIO.StringIO()
xls.save(output)

# Build an email
msg = MIMEMultipart()
msg['Subject'] = 'Hi, here is your file'
msg['From'] = email_from
msg['To'] = email_to
msg.preamble = 'Multipart message.\n'

# The attachment
part = MIMEApplication(output.getvalue())
part.add_header('Content-Disposition', 'attachment', filename=file_name)
part.add_header('Content-Type', 'text/csv; charset=UTF-8')
msg.attach(part)

# Connect to Amazon SES
ses = client(
    'ses',
    region_name='us-east-1',
    aws_access_key_id=config['AWS_ACCESS_KEY_ID'],
    aws_secret_access_key=config['AWS_SECRET_ACCESS_KEY'],
)
# And finally, send the email
ses.send_raw_email(
    Source=email_from,
    Destinations=[email_to],
    RawMessage={
        'Data': msg.as_string(),
    }
)
```