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

xls = xlwt.Workbook(style_compression=2)
sheet = xls.add_sheet('Sheet name')
header_style = xlwt.easyxf('font: bold True;')

# Header
labels = [
    u'Title 1',
    u'Title 2',
]
content = [labels]

for row in get_rows_with_content():
    content.append(list(row))

# Write it down
for row in range(len(content)):
    for col in range(len(content[row])):
        data = content[row][col]
        if row == 0:
            sheet.write(row, col, data, header_style)
            continue
        if type(data) == datetime.datetime:
            date_format = xlwt.XFStyle()
            data = data.replace(tzinfo=None)
            date_format.num_format_str = 'dd/mm/yyyy hh:mm:ss'
            sheet.write(row, col, data, date_format)
        else:
            sheet.write(row, col, data)

today = datetime.date.today()
file_name = 'my_file_{}.xls'.format(today.strftime("%Y-%m-%d"))

output = StringIO.StringIO()
xls.save(output)

msg = MIMEMultipart()
msg['Subject'] = 'Hi, here is your file {}'.format(today.strftime("%Y-%m-%d"))
msg['From'] = 'from@example.com'
msg['To'] = 'to@example.com'
msg.preamble = 'Multipart message.\n'

# The attachment
part = MIMEApplication(output.getvalue())
part.add_header('Content-Disposition', 'attachment', filename=file_name)
part.add_header('Content-Type', 'text/csv; charset=UTF-8')
msg.attach(part)

ses = client(
    'ses',
    region_name='us-east-1',
    aws_access_key_id=current_app.config['AWS_ACCESS_KEY_ID'],
    aws_secret_access_key=current_app.config['AWS_SECRET_ACCESS_KEY'],
)
ses.send_raw_email(
    Source=current_app.config['TAPPSI_DASHBOARD_EMAIL'],
    Destinations=[current_user.email],
    RawMessage={
        'Data': msg.as_string(),
    }
)
```