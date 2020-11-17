# QTM-350-Final-Project-
## Amazon Transcribe 
Amazon Transcribe is a AWS ML service that converts audio speech to text. Our project looks into the accurancy of the transcribe service.
To test the accurancy of Amazon Transcribe, we used songs given different speeds, levels of background noise, and artists' pronunciations. Then, we compared the lyrics transribed by Amazon Transcribed to the actual lyrics. 

#Using the AWS Transcribe from the SageMaker





#Option 1: Make S3 bucket and upload the music file to the bucket
We downloaded the mp3 file of the song called [Frozen | Let It Go](http://weathertron.keminglabs.com/p/let-it-go-lyrics-frozen-full-song-mp3-download/punpun).

import logging
import boto3
from botocore.exceptions import ClientError
import time
import json
import sys

#Make S3 bucket named tonno350
import boto3

s3_client = boto3.client('s3', region_name=None)
response = s3_client.create_bucket(
  Bucket='tonno350',
)


#upload .mp3 file 
s3 = boto3.client('s3')
filename = 'LetItGo.mp3'
bucket_name = 'tonno350'
s3.upload_file(filename, bucket_name, filename)

#save music file
import s3fs
fs = s3fs.S3FileSystem()
file = fs.open('s3://musicfortonno/LetItGo.mp3')


#Option 2: If already have a bucket and file uploaded
We have a bucket called `musicfortonno`. It is the file `LetItGo.mp3` seen below.

!aws s3 ls

!aws s3 ls musicfortonno

Next we create an instance of the transcribe client.

import wave
import asyncio
import boto3
import os
import time
import pandas as pd
import matplotlib as plt
from botocore.exceptions import ClientError
from datetime import date
import json
import seaborn as sns
import numpy as np
import time
import urllib

#transcribe Client
import boto3
import time
import urllib
import json

transcribe = boto3.client('transcribe')
transcribe

#AWS Transciption
Then, we follow the example provided by [AWS](https://docs.aws.amazon.com/transcribe/latest/dg/getting-started-python.html) for the transcription, but we made a few changes in here.

First, copy the S3 URI from the s3 console 

Then, in the code below, I replaced the string that was the value of the `job_uri` with the URL I copied, specifically ``s3://musicfortonno/LetItGo.mp`` and the transciption job will be saved as ``LetItGo``

#start transcription 
transcribe.start_transcription_job(
    TranscriptionJobName= "LetItGo",
    Media={'MediaFileUri': 's3://musicfortonno/LetItGo.mp3'},
    MediaFormat='mp3',
    LanguageCode='en-US'
)

To make things easier, we have changed ``LetItGo`` to ``job_name`` and ``'s3://musicfortonno/LetItGo.mp3'`` to ``job_uri``.

job_name = "LetItGo"
job_uri = 's3://musicfortonno/LetItGo.mp3'

Once the codes above finished, we can check the stauts if the transciption jobe is either completed, failed, or not ready yet.

There are two ways to check but both will generate the stauts.

#check status (1)
while True:
    status = transcribe.get_transcription_job(TranscriptionJobName=job_name)
    if status['TranscriptionJob']['TranscriptionJobStatus'] in ['COMPLETED', 'FAILED']:
        Break
    print("Not ready yet...")
    time.sleep(2)
print(status)

#check status (2) (preferred)
status = transcribe.get_transcription_job(TranscriptionJobName='LetItGo')
print(status)

Once the transcription job is compeleted, now we need to retrieve the transciption text and save it as .json file

We have derived the URI from ``TransciptionFileUri`` and we saved it as ``LetItGo.json``

URI for LetItGo is ``https://s3.us-east-1.amazonaws.com/aws-transcribe-us-east-1-prod/573540607053/LetItGo/25504118-af65-4db9-a814-4d5d393cedcd/asrOutput.json?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEKX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCbuq1BRG32PHaG3uiIKzQPZuqZ%2FS9uGrZxd5MtY7qcdQIgQzcVSJ6AOVnCwThPRacHlQ9c0xVKi82uaXF3EcEvvE8qtAMIHhACGgwyNzY2NTY0MzMxNTMiDK17W6UpMayAe7RSpCqRA%2BSFX03B7Cqy5abE7%2FIotFWWTn%2Ff0cjKSv7lLfbXezERv3kkPja7SE5c49TVUGkf2BPA6m4bclzgTWsnFqOM8KlySONLOVfuhoWNW28hBbetz%2B45CIYE6EiMZdBmjIW5YEyimz7Q%2FIudPemrMi%2FcawcSqjjUdDj8FFjKOr3jSPnhJ1DDiSpenquT1%2FIFKRu9742khsuhhThGPbpFnFLxeKzEIrQqcTDUqIoK0DSviMh51brRWi56HaijZqUHm9CTHvsWVnjAM6lVH5tSJz74ngOJ%2FCFpKrgHD%2FgVb6YxEqSpoaUjhn5s3xYTVNtlM1aXtMuviX61vUWdxq0cKfquLiCzO9jhjSitEk455%2Br1SYRqalTOFYKcqpoiX1Xl8vyqXRYDG16Wioef6qHOX2m2bXXgogSpDAzTCZ18hE76mONjFXBSLhKHy8IbMmm%2FUEjxhpqhK1FYSw3vEEIL5f85%2BHXuPjylHQD%2FytZ%2BRk8orkRDNYwznWJnmke8vGZgMzisfUNcaTyetvWKBnc8tVz9dvw6MLPGy%2F0FOusBI9hCeBx20JD9diCmBzvGmQQwxi2Xwcpe%2BLioU474yh%2BL5iNtaYCVmMP3ahwumRUp%2B3D6eqdj47La4ikyjmenJI5j%2Be0PYl37S3wERfhzIjZbNhrwVnnbGjoPHDruTLkW5RXzJDCRV36tysyawx5zer1Vym7mtdDaAoGxMylEf3e7gVydnyyNcnsDIpB1kzV8QKYap%2FRqdOW8UaK5YDLpxbuRGhF75psOLMP3u6g1zLHT68WkQOLjLrfGSMyobKiTLOIt%2F8Ef8zxwojv6m%2B9MP3f1agNxFqSKRUjg5VFBiKf8Ikp488lKP8t%2FDw%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20201116T221821Z&X-Amz-SignedHeaders=host&X-Amz-Expires=899&X-Amz-Credential=ASIAUA2QCFAA3YLWUVE3%2F20201116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=1a2cbd56d6aee7ba035fbdecd9a13b1b32c787d02432f2de5144ab9d4c956c10``

#Retrieve Text and save as ____.json
!wget -O LetItGo.json "https://s3.us-east-1.amazonaws.com/aws-transcribe-us-east-1-prod/573540607053/LetItGo/25504118-af65-4db9-a814-4d5d393cedcd/asrOutput.json?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEKX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCbuq1BRG32PHaG3uiIKzQPZuqZ%2FS9uGrZxd5MtY7qcdQIgQzcVSJ6AOVnCwThPRacHlQ9c0xVKi82uaXF3EcEvvE8qtAMIHhACGgwyNzY2NTY0MzMxNTMiDK17W6UpMayAe7RSpCqRA%2BSFX03B7Cqy5abE7%2FIotFWWTn%2Ff0cjKSv7lLfbXezERv3kkPja7SE5c49TVUGkf2BPA6m4bclzgTWsnFqOM8KlySONLOVfuhoWNW28hBbetz%2B45CIYE6EiMZdBmjIW5YEyimz7Q%2FIudPemrMi%2FcawcSqjjUdDj8FFjKOr3jSPnhJ1DDiSpenquT1%2FIFKRu9742khsuhhThGPbpFnFLxeKzEIrQqcTDUqIoK0DSviMh51brRWi56HaijZqUHm9CTHvsWVnjAM6lVH5tSJz74ngOJ%2FCFpKrgHD%2FgVb6YxEqSpoaUjhn5s3xYTVNtlM1aXtMuviX61vUWdxq0cKfquLiCzO9jhjSitEk455%2Br1SYRqalTOFYKcqpoiX1Xl8vyqXRYDG16Wioef6qHOX2m2bXXgogSpDAzTCZ18hE76mONjFXBSLhKHy8IbMmm%2FUEjxhpqhK1FYSw3vEEIL5f85%2BHXuPjylHQD%2FytZ%2BRk8orkRDNYwznWJnmke8vGZgMzisfUNcaTyetvWKBnc8tVz9dvw6MLPGy%2F0FOusBI9hCeBx20JD9diCmBzvGmQQwxi2Xwcpe%2BLioU474yh%2BL5iNtaYCVmMP3ahwumRUp%2B3D6eqdj47La4ikyjmenJI5j%2Be0PYl37S3wERfhzIjZbNhrwVnnbGjoPHDruTLkW5RXzJDCRV36tysyawx5zer1Vym7mtdDaAoGxMylEf3e7gVydnyyNcnsDIpB1kzV8QKYap%2FRqdOW8UaK5YDLpxbuRGhF75psOLMP3u6g1zLHT68WkQOLjLrfGSMyobKiTLOIt%2F8Ef8zxwojv6m%2B9MP3f1agNxFqSKRUjg5VFBiKf8Ikp488lKP8t%2FDw%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20201116T221821Z&X-Amz-SignedHeaders=host&X-Amz-Expires=899&X-Amz-Credential=ASIAUA2QCFAA3YLWUVE3%2F20201116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=1a2cbd56d6aee7ba035fbdecd9a13b1b32c787d02432f2de5144ab9d4c956c10"

Now we are checking if we have the file downloaded

#checking we have .json file
!ls

To see the file

#Show .json
!cat LetItGo.json

If we want to change the ``.json`` file either to ``.docx`` or ``.csv`` to generate the data frame

#change .json to .docx
import tscribe
tscribe.write("asrOutput.json")

#change .json to .csv
tscribe.write("asrOutput.json", format="csv", save_as="LetItGo_Disney.csv")

Above described the basic steps to use the SageMaker to do AWS Transcribe and generate ``.json`` file at the end.
