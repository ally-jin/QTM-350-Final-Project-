# QTM-350-Final-Project-
## Amazon Transcribe 
Amazon Transcribe is a AWS ML service that converts audio speech to text. Our project looks into the accurancy of the transcribe service.
To test the accurancy of Amazon Transcribe, we used songs given different speeds, levels of background noise, and artists' pronunciations. Then, we compared the lyrics transribed by Amazon Transcribed to the actual lyrics. 

{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "kernelspec": {
      "display_name": "conda_python3",
      "language": "python",
      "name": "conda_python3"
    },
    "language_info": {
      "codemirror_mode": {
        "name": "ipython",
        "version": 3
      },
      "file_extension": ".py",
      "mimetype": "text/x-python",
      "name": "python",
      "nbconvert_exporter": "python",
      "pygments_lexer": "ipython3",
      "version": "3.6.10"
    },
    "colab": {
      "name": "QTM350_FinalProject_AWSTranscribe_fromSageMaker.ipynb",
      "provenance": [],
      "collapsed_sections": []
    }
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "0Q3VFO0NpCD0"
      },
      "source": [
        "#Using the AWS Transcribe from the SageMaker\n",
        "\n",
        "\n",
        "\n"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "ndkKINxCowVW"
      },
      "source": [
        "#Option 1: Make S3 bucket and upload the music file to the bucket\n",
        "We downloaded the mp3 file of the song called [Frozen | Let It Go](http://weathertron.keminglabs.com/p/let-it-go-lyrics-frozen-full-song-mp3-download/punpun)."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "4AG9tSfRogAu"
      },
      "source": [
        "import logging\n",
        "import boto3\n",
        "from botocore.exceptions import ClientError\n",
        "import time\n",
        "import json\n",
        "import sys"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "VVADKTwVogAu"
      },
      "source": [
        "#Make S3 bucket named tonno350\n",
        "import boto3\n",
        "\n",
        "s3_client = boto3.client('s3', region_name=None)\n",
        "response = s3_client.create_bucket(\n",
        "  Bucket='tonno350',\n",
        ")\n"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "VtFn6tPjogAv"
      },
      "source": [
        "#upload .mp3 file \n",
        "s3 = boto3.client('s3')\n",
        "filename = 'LetItGo.mp3'\n",
        "bucket_name = 'tonno350'\n",
        "s3.upload_file(filename, bucket_name, filename)"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "obdB-X0bogAz"
      },
      "source": [
        "#save music file\n",
        "import s3fs\n",
        "fs = s3fs.S3FileSystem()\n",
        "file = fs.open('s3://musicfortonno/LetItGo.mp3')"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "eHGXi7J2ok4D"
      },
      "source": [
        "\n",
        "#Option 2: If already have a bucket and file uploaded\n",
        "We have a bucket called `musicfortonno`. It is the file `LetItGo.mp3` seen below."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "PjKcxqmlogAr",
        "outputId": "1eac5579-9e9f-42bf-a525-d6bf525aa650"
      },
      "source": [
        "!aws s3 ls"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "2020-11-02 23:06:58 musicfortonno\n",
            "2020-10-22 22:05:00 notebook-hosting-bucket-mina\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "SQe8E5auogAu",
        "outputId": "15c51e90-ade4-4364-df60-7df24d54fdc6"
      },
      "source": [
        "!aws s3 ls musicfortonno"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "                           PRE music.json/\n",
            "2020-11-03 17:04:28     643488 5fa18cdbd41e7-halomp3.vocal.mp3\n",
            "2020-11-03 17:43:08    6711516 Adele - Someone Like You (Lyrics).mp3\n",
            "2020-11-03 17:43:09    6225012 Adele - Someone Like You (Official Studio Acapella).mp3\n",
            "2020-11-03 16:27:26    5735999 Beyoncé - Halo (Studio Acapella).mp3\n",
            "2020-11-03 17:12:47    5356075 Bruno Mars - Just the Way You Are (Vocal Track Only).mp3\n",
            "2020-11-03 01:10:15    8338329 Dynamite-Taio Cruz.mp3\n",
            "2020-11-03 16:18:39    6282063 Halo.mp3\n",
            "2020-11-03 17:12:47    5317831 Just The Way You Are - Bruno Mars (Lyrics).mp3\n",
            "2020-11-03 17:12:47    5294635 Just the Way You Are (No Drums) [Lyric Video] - Bruno Mars.mp3\n",
            "2020-11-03 17:56:18    5657005 Let It Go (No Music, Just Vocals and Realistic Sounds).mp3\n",
            "2020-11-16 19:44:29    5374883 LetItGo.mp3\n",
            "2020-11-03 16:51:49    4684623 [MR Removed 엠알 제거] 방탄소년단(BTS) - 다이너마이트(DYNAMITE).mp3\n",
            "2020-11-03 17:56:18    4796845 [MR-RemovedAcapella] Idina Menzel - Let It Go All Vocal.mp3\n",
            "2020-11-02 23:13:46    3100141 아웃사이더 - 외톨이 (128  kbps) (abdwap2.com).mp3\n",
            "2020-11-03 00:43:52    8633084 아웃사이더 외톨이(0.5배속).mp3\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "6g-ZDZigp2sK"
      },
      "source": [
        "Next we create an instance of the transcribe client."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "QAAgGoudogAx"
      },
      "source": [
        "import wave\n",
        "import asyncio\n",
        "import boto3\n",
        "import os\n",
        "import time\n",
        "import pandas as pd\n",
        "import matplotlib as plt\n",
        "from botocore.exceptions import ClientError\n",
        "from datetime import date\n",
        "import json\n",
        "import seaborn as sns\n",
        "import numpy as np\n",
        "import time\n",
        "import urllib"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "l26lERdRogAx",
        "outputId": "5778a8b0-6e21-44fd-80b3-b9d986af56f6"
      },
      "source": [
        "#transcribe Client\n",
        "import boto3\n",
        "import time\n",
        "import urllib\n",
        "import json\n",
        "\n",
        "transcribe = boto3.client('transcribe')\n",
        "transcribe"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "<botocore.client.TranscribeService at 0x7fe180a909e8>"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 33
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "jZJv-NeEqEKI"
      },
      "source": [
        "#AWS Transciption\n",
        "Then, we follow the example provided by [AWS](https://docs.aws.amazon.com/transcribe/latest/dg/getting-started-python.html) for the transcription, but we made a few changes in here.\n",
        "\n",
        "First, copy the S3 URI from the s3 console \n",
        "\n",
        "Then, in the code below, I replaced the string that was the value of the `job_uri` with the URL I copied, specifically ``s3://musicfortonno/LetItGo.mp`` and the transciption job will be saved as ``LetItGo``"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "5SIQCydUogAx",
        "outputId": "24362f2f-62fb-4963-849a-02bbd54467e3"
      },
      "source": [
        "#start transcription \n",
        "transcribe.start_transcription_job(\n",
        "    TranscriptionJobName= \"LetItGo\",\n",
        "    Media={'MediaFileUri': 's3://musicfortonno/LetItGo.mp3'},\n",
        "    MediaFormat='mp3',\n",
        "    LanguageCode='en-US'\n",
        ")"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "{'TranscriptionJob': {'TranscriptionJobName': 'LetItGo',\n",
              "  'TranscriptionJobStatus': 'IN_PROGRESS',\n",
              "  'LanguageCode': 'en-US',\n",
              "  'MediaFormat': 'mp3',\n",
              "  'Media': {'MediaFileUri': 's3://musicfortonno/LetItGo.mp3'},\n",
              "  'StartTime': datetime.datetime(2020, 11, 16, 22, 16, 4, 534000, tzinfo=tzlocal()),\n",
              "  'CreationTime': datetime.datetime(2020, 11, 16, 22, 16, 4, 492000, tzinfo=tzlocal())},\n",
              " 'ResponseMetadata': {'RequestId': '934fb201-4b98-432f-b7c4-23bb2b051cf1',\n",
              "  'HTTPStatusCode': 200,\n",
              "  'HTTPHeaders': {'content-type': 'application/x-amz-json-1.1',\n",
              "   'date': 'Mon, 16 Nov 2020 22:16:04 GMT',\n",
              "   'x-amzn-requestid': '934fb201-4b98-432f-b7c4-23bb2b051cf1',\n",
              "   'content-length': '256',\n",
              "   'connection': 'keep-alive'},\n",
              "  'RetryAttempts': 0}}"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 36
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "LlmyrfcLr_jX"
      },
      "source": [
        "To make things easier, we have changed ``LetItGo`` to ``job_name`` and ``'s3://musicfortonno/LetItGo.mp3'`` to ``job_uri``."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "fVCydvweogAx"
      },
      "source": [
        "job_name = \"LetItGo\"\n",
        "job_uri = 's3://musicfortonno/LetItGo.mp3'"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "rxlBgOtOsIFp"
      },
      "source": [
        "Once the codes above finished, we can check the stauts if the transciption jobe is either completed, failed, or not ready yet.\n",
        "\n",
        "There are two ways to check but both will generate the stauts."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "vf_XHPrNogAx"
      },
      "source": [
        "#check status (1)\n",
        "while True:\n",
        "    status = transcribe.get_transcription_job(TranscriptionJobName=job_name)\n",
        "    if status['TranscriptionJob']['TranscriptionJobStatus'] in ['COMPLETED', 'FAILED']:\n",
        "        Break\n",
        "    print(\"Not ready yet...\")\n",
        "    time.sleep(2)\n",
        "print(status)"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "1eOx3ME5ogAx",
        "outputId": "e1b7ae4b-755d-4539-bd1f-8ef261917630"
      },
      "source": [
        "#check status (2) (preferred)\n",
        "status = transcribe.get_transcription_job(TranscriptionJobName='LetItGo')\n",
        "print(status)"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "{'TranscriptionJob': {'TranscriptionJobName': 'LetItGo', 'TranscriptionJobStatus': 'COMPLETED', 'LanguageCode': 'en-US', 'MediaSampleRateHertz': 44100, 'MediaFormat': 'mp3', 'Media': {'MediaFileUri': 's3://musicfortonno/LetItGo.mp3'}, 'Transcript': {'TranscriptFileUri': 'https://s3.us-east-1.amazonaws.com/aws-transcribe-us-east-1-prod/573540607053/LetItGo/25504118-af65-4db9-a814-4d5d393cedcd/asrOutput.json?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEKX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCbuq1BRG32PHaG3uiIKzQPZuqZ%2FS9uGrZxd5MtY7qcdQIgQzcVSJ6AOVnCwThPRacHlQ9c0xVKi82uaXF3EcEvvE8qtAMIHhACGgwyNzY2NTY0MzMxNTMiDK17W6UpMayAe7RSpCqRA%2BSFX03B7Cqy5abE7%2FIotFWWTn%2Ff0cjKSv7lLfbXezERv3kkPja7SE5c49TVUGkf2BPA6m4bclzgTWsnFqOM8KlySONLOVfuhoWNW28hBbetz%2B45CIYE6EiMZdBmjIW5YEyimz7Q%2FIudPemrMi%2FcawcSqjjUdDj8FFjKOr3jSPnhJ1DDiSpenquT1%2FIFKRu9742khsuhhThGPbpFnFLxeKzEIrQqcTDUqIoK0DSviMh51brRWi56HaijZqUHm9CTHvsWVnjAM6lVH5tSJz74ngOJ%2FCFpKrgHD%2FgVb6YxEqSpoaUjhn5s3xYTVNtlM1aXtMuviX61vUWdxq0cKfquLiCzO9jhjSitEk455%2Br1SYRqalTOFYKcqpoiX1Xl8vyqXRYDG16Wioef6qHOX2m2bXXgogSpDAzTCZ18hE76mONjFXBSLhKHy8IbMmm%2FUEjxhpqhK1FYSw3vEEIL5f85%2BHXuPjylHQD%2FytZ%2BRk8orkRDNYwznWJnmke8vGZgMzisfUNcaTyetvWKBnc8tVz9dvw6MLPGy%2F0FOusBI9hCeBx20JD9diCmBzvGmQQwxi2Xwcpe%2BLioU474yh%2BL5iNtaYCVmMP3ahwumRUp%2B3D6eqdj47La4ikyjmenJI5j%2Be0PYl37S3wERfhzIjZbNhrwVnnbGjoPHDruTLkW5RXzJDCRV36tysyawx5zer1Vym7mtdDaAoGxMylEf3e7gVydnyyNcnsDIpB1kzV8QKYap%2FRqdOW8UaK5YDLpxbuRGhF75psOLMP3u6g1zLHT68WkQOLjLrfGSMyobKiTLOIt%2F8Ef8zxwojv6m%2B9MP3f1agNxFqSKRUjg5VFBiKf8Ikp488lKP8t%2FDw%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20201116T221821Z&X-Amz-SignedHeaders=host&X-Amz-Expires=899&X-Amz-Credential=ASIAUA2QCFAA3YLWUVE3%2F20201116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=1a2cbd56d6aee7ba035fbdecd9a13b1b32c787d02432f2de5144ab9d4c956c10'}, 'StartTime': datetime.datetime(2020, 11, 16, 22, 16, 4, 534000, tzinfo=tzlocal()), 'CreationTime': datetime.datetime(2020, 11, 16, 22, 16, 4, 492000, tzinfo=tzlocal()), 'CompletionTime': datetime.datetime(2020, 11, 16, 22, 17, 40, 94000, tzinfo=tzlocal()), 'Settings': {'ChannelIdentification': False, 'ShowAlternatives': False}}, 'ResponseMetadata': {'RequestId': '67392d48-8f5d-47a3-8921-371e71cee56a', 'HTTPStatusCode': 200, 'HTTPHeaders': {'content-type': 'application/x-amz-json-1.1', 'date': 'Mon, 16 Nov 2020 22:18:21 GMT', 'x-amzn-requestid': '67392d48-8f5d-47a3-8921-371e71cee56a', 'content-length': '1976', 'connection': 'keep-alive'}, 'RetryAttempts': 0}}\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "bfO5Z38DsiXz"
      },
      "source": [
        "Once the transcription job is compeleted, now we need to retrieve the transciption text and save it as .json file\n",
        "\n",
        "We have derived the URI from ``TransciptionFileUri`` and we saved it as ``LetItGo.json``\n",
        "\n",
        "URI for LetItGo is ``https://s3.us-east-1.amazonaws.com/aws-transcribe-us-east-1-prod/573540607053/LetItGo/25504118-af65-4db9-a814-4d5d393cedcd/asrOutput.json?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEKX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCbuq1BRG32PHaG3uiIKzQPZuqZ%2FS9uGrZxd5MtY7qcdQIgQzcVSJ6AOVnCwThPRacHlQ9c0xVKi82uaXF3EcEvvE8qtAMIHhACGgwyNzY2NTY0MzMxNTMiDK17W6UpMayAe7RSpCqRA%2BSFX03B7Cqy5abE7%2FIotFWWTn%2Ff0cjKSv7lLfbXezERv3kkPja7SE5c49TVUGkf2BPA6m4bclzgTWsnFqOM8KlySONLOVfuhoWNW28hBbetz%2B45CIYE6EiMZdBmjIW5YEyimz7Q%2FIudPemrMi%2FcawcSqjjUdDj8FFjKOr3jSPnhJ1DDiSpenquT1%2FIFKRu9742khsuhhThGPbpFnFLxeKzEIrQqcTDUqIoK0DSviMh51brRWi56HaijZqUHm9CTHvsWVnjAM6lVH5tSJz74ngOJ%2FCFpKrgHD%2FgVb6YxEqSpoaUjhn5s3xYTVNtlM1aXtMuviX61vUWdxq0cKfquLiCzO9jhjSitEk455%2Br1SYRqalTOFYKcqpoiX1Xl8vyqXRYDG16Wioef6qHOX2m2bXXgogSpDAzTCZ18hE76mONjFXBSLhKHy8IbMmm%2FUEjxhpqhK1FYSw3vEEIL5f85%2BHXuPjylHQD%2FytZ%2BRk8orkRDNYwznWJnmke8vGZgMzisfUNcaTyetvWKBnc8tVz9dvw6MLPGy%2F0FOusBI9hCeBx20JD9diCmBzvGmQQwxi2Xwcpe%2BLioU474yh%2BL5iNtaYCVmMP3ahwumRUp%2B3D6eqdj47La4ikyjmenJI5j%2Be0PYl37S3wERfhzIjZbNhrwVnnbGjoPHDruTLkW5RXzJDCRV36tysyawx5zer1Vym7mtdDaAoGxMylEf3e7gVydnyyNcnsDIpB1kzV8QKYap%2FRqdOW8UaK5YDLpxbuRGhF75psOLMP3u6g1zLHT68WkQOLjLrfGSMyobKiTLOIt%2F8Ef8zxwojv6m%2B9MP3f1agNxFqSKRUjg5VFBiKf8Ikp488lKP8t%2FDw%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20201116T221821Z&X-Amz-SignedHeaders=host&X-Amz-Expires=899&X-Amz-Credential=ASIAUA2QCFAA3YLWUVE3%2F20201116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=1a2cbd56d6aee7ba035fbdecd9a13b1b32c787d02432f2de5144ab9d4c956c10``"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "nn8k3t37ogAx",
        "outputId": "53ae4463-05ba-4612-c35f-456dc5a36b13"
      },
      "source": [
        "#Retrieve Text and save as ____.json\n",
        "!wget -O LetItGo.json \"https://s3.us-east-1.amazonaws.com/aws-transcribe-us-east-1-prod/573540607053/LetItGo/25504118-af65-4db9-a814-4d5d393cedcd/asrOutput.json?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEKX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCbuq1BRG32PHaG3uiIKzQPZuqZ%2FS9uGrZxd5MtY7qcdQIgQzcVSJ6AOVnCwThPRacHlQ9c0xVKi82uaXF3EcEvvE8qtAMIHhACGgwyNzY2NTY0MzMxNTMiDK17W6UpMayAe7RSpCqRA%2BSFX03B7Cqy5abE7%2FIotFWWTn%2Ff0cjKSv7lLfbXezERv3kkPja7SE5c49TVUGkf2BPA6m4bclzgTWsnFqOM8KlySONLOVfuhoWNW28hBbetz%2B45CIYE6EiMZdBmjIW5YEyimz7Q%2FIudPemrMi%2FcawcSqjjUdDj8FFjKOr3jSPnhJ1DDiSpenquT1%2FIFKRu9742khsuhhThGPbpFnFLxeKzEIrQqcTDUqIoK0DSviMh51brRWi56HaijZqUHm9CTHvsWVnjAM6lVH5tSJz74ngOJ%2FCFpKrgHD%2FgVb6YxEqSpoaUjhn5s3xYTVNtlM1aXtMuviX61vUWdxq0cKfquLiCzO9jhjSitEk455%2Br1SYRqalTOFYKcqpoiX1Xl8vyqXRYDG16Wioef6qHOX2m2bXXgogSpDAzTCZ18hE76mONjFXBSLhKHy8IbMmm%2FUEjxhpqhK1FYSw3vEEIL5f85%2BHXuPjylHQD%2FytZ%2BRk8orkRDNYwznWJnmke8vGZgMzisfUNcaTyetvWKBnc8tVz9dvw6MLPGy%2F0FOusBI9hCeBx20JD9diCmBzvGmQQwxi2Xwcpe%2BLioU474yh%2BL5iNtaYCVmMP3ahwumRUp%2B3D6eqdj47La4ikyjmenJI5j%2Be0PYl37S3wERfhzIjZbNhrwVnnbGjoPHDruTLkW5RXzJDCRV36tysyawx5zer1Vym7mtdDaAoGxMylEf3e7gVydnyyNcnsDIpB1kzV8QKYap%2FRqdOW8UaK5YDLpxbuRGhF75psOLMP3u6g1zLHT68WkQOLjLrfGSMyobKiTLOIt%2F8Ef8zxwojv6m%2B9MP3f1agNxFqSKRUjg5VFBiKf8Ikp488lKP8t%2FDw%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20201116T221821Z&X-Amz-SignedHeaders=host&X-Amz-Expires=899&X-Amz-Credential=ASIAUA2QCFAA3YLWUVE3%2F20201116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=1a2cbd56d6aee7ba035fbdecd9a13b1b32c787d02432f2de5144ab9d4c956c10\""
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "--2020-11-16 22:20:26--  https://s3.us-east-1.amazonaws.com/aws-transcribe-us-east-1-prod/573540607053/LetItGo/25504118-af65-4db9-a814-4d5d393cedcd/asrOutput.json?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEKX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCbuq1BRG32PHaG3uiIKzQPZuqZ%2FS9uGrZxd5MtY7qcdQIgQzcVSJ6AOVnCwThPRacHlQ9c0xVKi82uaXF3EcEvvE8qtAMIHhACGgwyNzY2NTY0MzMxNTMiDK17W6UpMayAe7RSpCqRA%2BSFX03B7Cqy5abE7%2FIotFWWTn%2Ff0cjKSv7lLfbXezERv3kkPja7SE5c49TVUGkf2BPA6m4bclzgTWsnFqOM8KlySONLOVfuhoWNW28hBbetz%2B45CIYE6EiMZdBmjIW5YEyimz7Q%2FIudPemrMi%2FcawcSqjjUdDj8FFjKOr3jSPnhJ1DDiSpenquT1%2FIFKRu9742khsuhhThGPbpFnFLxeKzEIrQqcTDUqIoK0DSviMh51brRWi56HaijZqUHm9CTHvsWVnjAM6lVH5tSJz74ngOJ%2FCFpKrgHD%2FgVb6YxEqSpoaUjhn5s3xYTVNtlM1aXtMuviX61vUWdxq0cKfquLiCzO9jhjSitEk455%2Br1SYRqalTOFYKcqpoiX1Xl8vyqXRYDG16Wioef6qHOX2m2bXXgogSpDAzTCZ18hE76mONjFXBSLhKHy8IbMmm%2FUEjxhpqhK1FYSw3vEEIL5f85%2BHXuPjylHQD%2FytZ%2BRk8orkRDNYwznWJnmke8vGZgMzisfUNcaTyetvWKBnc8tVz9dvw6MLPGy%2F0FOusBI9hCeBx20JD9diCmBzvGmQQwxi2Xwcpe%2BLioU474yh%2BL5iNtaYCVmMP3ahwumRUp%2B3D6eqdj47La4ikyjmenJI5j%2Be0PYl37S3wERfhzIjZbNhrwVnnbGjoPHDruTLkW5RXzJDCRV36tysyawx5zer1Vym7mtdDaAoGxMylEf3e7gVydnyyNcnsDIpB1kzV8QKYap%2FRqdOW8UaK5YDLpxbuRGhF75psOLMP3u6g1zLHT68WkQOLjLrfGSMyobKiTLOIt%2F8Ef8zxwojv6m%2B9MP3f1agNxFqSKRUjg5VFBiKf8Ikp488lKP8t%2FDw%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20201116T221821Z&X-Amz-SignedHeaders=host&X-Amz-Expires=899&X-Amz-Credential=ASIAUA2QCFAA3YLWUVE3%2F20201116%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=1a2cbd56d6aee7ba035fbdecd9a13b1b32c787d02432f2de5144ab9d4c956c10\n",
            "Resolving s3.us-east-1.amazonaws.com (s3.us-east-1.amazonaws.com)... 52.217.12.86\n",
            "Connecting to s3.us-east-1.amazonaws.com (s3.us-east-1.amazonaws.com)|52.217.12.86|:443... connected.\n",
            "HTTP request sent, awaiting response... 200 OK\n",
            "Length: 23754 (23K) [application/octet-stream]\n",
            "Saving to: ‘LetItGo.json’\n",
            "\n",
            "LetItGo.json        100%[===================>]  23.20K  --.-KB/s    in 0.002s  \n",
            "\n",
            "2020-11-16 22:20:26 (13.6 MB/s) - ‘LetItGo.json’ saved [23754/23754]\n",
            "\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "5IuOkb25tRAw"
      },
      "source": [
        "Now we are checking if we have the file downloaded"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "ohSs7Y0VogAz",
        "outputId": "c6e294ef-6f43-4279-b276-7b4f6a8bccd2"
      },
      "source": [
        "#checking we have .json file\n",
        "!ls"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "asrOutput.json\n",
            "asrOutput.json?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEKX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCbuq1BRG32PHaG3uiIKzQPZuqZ%2FS9uGrZxd5MtY7qcdQIgQzcVSJ6AOVnCwThPRacHlQ9c0xVKi82uaXF3EcEvvE8qtAMIHhACGgwyNzY2NTY0MzMxNTMiDK17W6\n",
            "awsclibundle.zip\n",
            "chart.png\n",
            "degree.csv\n",
            "example.json\n",
            "example-transcribe-notebook (1).ipynb\n",
            "exercise1-mina.sh\n",
            "FinalProject.ipynb\n",
            "LetItGo_Disney.csv\n",
            "LetItGo_Disney.docx\n",
            "LetItGo.json\n",
            "LetItGo_original.json\n",
            "LetItGo.txt\n",
            "lost+found\n",
            "__MACOSX\n",
            "mina\n",
            "order-<mina>.sh\n",
            "positional.sh \n",
            "Quiz10Q7.ipynb\n",
            "Tonno_HW2.html\n",
            "Tonno_HW2.ipynb\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "1ZEZLSTqtbPK"
      },
      "source": [
        "To see the file"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "afmcQOg1ogAz",
        "outputId": "11fbfda6-85a7-4827-fadf-faf269e0836a"
      },
      "source": [
        "#Show .json\n",
        "!cat LetItGo.json"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "{\"jobName\":\"LetItGo\",\"accountId\":\"573540607053\",\"results\":{\"transcripts\":[{\"transcript\":\"snow glows white on the mountain tonight Not a footprint to be seen A kingdom of isolation And it looks like I'm the queen. The wind is howling like this swirling storm inside Couldn't keep it and have a nose I don't let them in Don't let them see how be the good girl. You always have to be concealed Don't feel don't let them know Well, now that we Oh, let it go Can't hold it back anymore Let it go Let it go Turn away and slam e o care what they're going Thio He let the store Ragen cold Never bothered me anyway. Way It's funny how some distance makes everything seem smooth Fears and one's controlled me Can't get Thio See what I can do Test the list Mm It's and you know I know wrote no booze for me e o oh, let me Oh, you know my power flurries through the air into that around my sword They spiraling in frozen froth on the way back Past is in the Oh, Cole never bothered me anyway\"}],\"items\":[{\"start_time\":\"14.11\",\"end_time\":\"14.72\",\"alternatives\":[{\"confidence\":\"0.9995\",\"content\":\"snow\"}],\"type\":\"pronunciation\"},{\"start_time\":\"14.72\",\"end_time\":\"15.15\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"glows\"}],\"type\":\"pronunciation\"},{\"start_time\":\"15.15\",\"end_time\":\"15.7\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"white\"}],\"type\":\"pronunciation\"},{\"start_time\":\"15.7\",\"end_time\":\"15.85\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"on\"}],\"type\":\"pronunciation\"},{\"start_time\":\"15.85\",\"end_time\":\"15.95\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"the\"}],\"type\":\"pronunciation\"},{\"start_time\":\"15.95\",\"end_time\":\"16.44\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"mountain\"}],\"type\":\"pronunciation\"},{\"start_time\":\"16.44\",\"end_time\":\"17.2\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"tonight\"}],\"type\":\"pronunciation\"},{\"start_time\":\"17.27\",\"end_time\":\"17.61\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"Not\"}],\"type\":\"pronunciation\"},{\"start_time\":\"17.61\",\"end_time\":\"17.68\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"a\"}],\"type\":\"pronunciation\"},{\"start_time\":\"17.68\",\"end_time\":\"18.76\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"footprint\"}],\"type\":\"pronunciation\"},{\"start_time\":\"18.77\",\"end_time\":\"18.99\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"to\"}],\"type\":\"pronunciation\"},{\"start_time\":\"18.99\",\"end_time\":\"19.12\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"be\"}],\"type\":\"pronunciation\"},{\"start_time\":\"19.12\",\"end_time\":\"20.65\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"seen\"}],\"type\":\"pronunciation\"},{\"start_time\":\"21.04\",\"end_time\":\"21.23\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"A\"}],\"type\":\"pronunciation\"},{\"start_time\":\"21.24\",\"end_time\":\"21.79\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"kingdom\"}],\"type\":\"pronunciation\"},{\"start_time\":\"21.79\",\"end_time\":\"22.05\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"of\"}],\"type\":\"pronunciation\"},{\"start_time\":\"22.05\",\"end_time\":\"23.77\",\"alternatives\":[{\"confidence\":\"0.9995\",\"content\":\"isolation\"}],\"type\":\"pronunciation\"},{\"start_time\":\"24.24\",\"end_time\":\"24.61\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"And\"}],\"type\":\"pronunciation\"},{\"start_time\":\"24.61\",\"end_time\":\"24.74\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"it\"}],\"type\":\"pronunciation\"},{\"start_time\":\"24.74\",\"end_time\":\"25.09\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"looks\"}],\"type\":\"pronunciation\"},{\"start_time\":\"25.09\",\"end_time\":\"25.9\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"like\"}],\"type\":\"pronunciation\"},{\"start_time\":\"25.91\",\"end_time\":\"26.15\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"I'm\"}],\"type\":\"pronunciation\"},{\"start_time\":\"26.15\",\"end_time\":\"26.24\",\"alternatives\":[{\"confidence\":\"0.9995\",\"content\":\"the\"}],\"type\":\"pronunciation\"},{\"start_time\":\"26.25\",\"end_time\":\"27.27\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"queen\"}],\"type\":\"pronunciation\"},{\"alternatives\":[{\"confidence\":\"0.0\",\"content\":\".\"}],\"type\":\"punctuation\"},{\"start_time\":\"28.24\",\"end_time\":\"28.31\",\"alternatives\":[{\"confidence\":\"0.7543\",\"content\":\"The\"}],\"type\":\"pronunciation\"},{\"start_time\":\"28.78\",\"end_time\":\"29.48\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"wind\"}],\"type\":\"pronunciation\"},{\"start_time\":\"29.48\",\"end_time\":\"29.85\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"is\"}],\"type\":\"pronunciation\"},{\"start_time\":\"29.85\",\"end_time\":\"30.71\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"howling\"}],\"type\":\"pronunciation\"},{\"start_time\":\"30.72\",\"end_time\":\"31.22\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"like\"}],\"type\":\"pronunciation\"},{\"start_time\":\"31.23\",\"end_time\":\"31.52\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"this\"}],\"type\":\"pronunciation\"},{\"start_time\":\"31.52\",\"end_time\":\"32.42\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"swirling\"}],\"type\":\"pronunciation\"},{\"start_time\":\"32.42\",\"end_time\":\"33.07\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"storm\"}],\"type\":\"pronunciation\"},{\"start_time\":\"33.07\",\"end_time\":\"35.05\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"inside\"}],\"type\":\"pronunciation\"},{\"start_time\":\"35.58\",\"end_time\":\"36.13\",\"alternatives\":[{\"confidence\":\"0.5534\",\"content\":\"Couldn't\"}],\"type\":\"pronunciation\"},{\"start_time\":\"36.14\",\"end_time\":\"36.49\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"keep\"}],\"type\":\"pronunciation\"},{\"start_time\":\"36.49\",\"end_time\":\"36.86\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"it\"}],\"type\":\"pronunciation\"},{\"start_time\":\"36.86\",\"end_time\":\"37.12\",\"alternatives\":[{\"confidence\":\"0.9994\",\"content\":\"and\"}],\"type\":\"pronunciation\"},{\"start_time\":\"37.13\",\"end_time\":\"37.69\",\"alternatives\":[{\"confidence\":\"0.8251\",\"content\":\"have\"}],\"type\":\"pronunciation\"},{\"start_time\":\"37.69\",\"end_time\":\"37.76\",\"alternatives\":[{\"confidence\":\"0.8251\",\"content\":\"a\"}],\"type\":\"pronunciation\"},{\"start_time\":\"37.76\",\"end_time\":\"38.42\",\"alternatives\":[{\"confidence\":\"0.672\",\"content\":\"nose\"}],\"type\":\"pronunciation\"},{\"start_time\":\"38.42\",\"end_time\":\"38.89\",\"alternatives\":[{\"confidence\":\"0.999\",\"content\":\"I\"}],\"type\":\"pronunciation\"},{\"start_time\":\"42.43\",\"end_time\":\"43.05\",\"alternatives\":[{\"confidence\":\"0.9836\",\"content\":\"don't\"}],\"type\":\"pronunciation\"},{\"start_time\":\"43.05\",\"end_time\":\"43.4\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"let\"}],\"type\":\"pronunciation\"},{\"start_time\":\"43.41\",\"end_time\":\"43.87\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"them\"}],\"type\":\"pronunciation\"},{\"start_time\":\"43.87\",\"end_time\":\"44.14\",\"alternatives\":[{\"confidence\":\"0.9995\",\"content\":\"in\"}],\"type\":\"pronunciation\"},{\"start_time\":\"44.15\",\"end_time\":\"44.69\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"Don't\"}],\"type\":\"pronunciation\"},{\"start_time\":\"44.69\",\"end_time\":\"45.13\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"let\"}],\"type\":\"pronunciation\"},{\"start_time\":\"45.14\",\"end_time\":\"45.47\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"them\"}],\"type\":\"pronunciation\"},{\"start_time\":\"45.47\",\"end_time\":\"45.83\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"see\"}],\"type\":\"pronunciation\"},{\"start_time\":\"45.83\",\"end_time\":\"45.97\",\"alternatives\":[{\"confidence\":\"0.8608\",\"content\":\"how\"}],\"type\":\"pronunciation\"},{\"start_time\":\"46.04\",\"end_time\":\"46.45\",\"alternatives\":[{\"confidence\":\"0.9539\",\"content\":\"be\"}],\"type\":\"pronunciation\"},{\"start_time\":\"46.45\",\"end_time\":\"46.6\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"the\"}],\"type\":\"pronunciation\"},{\"start_time\":\"46.61\",\"end_time\":\"46.88\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"good\"}],\"type\":\"pronunciation\"},{\"start_time\":\"46.88\",\"end_time\":\"47.32\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"girl\"}],\"type\":\"pronunciation\"},{\"alternatives\":[{\"confidence\":\"0.0\",\"content\":\".\"}],\"type\":\"punctuation\"},{\"start_time\":\"47.33\",\"end_time\":\"47.49\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"You\"}],\"type\":\"pronunciation\"},{\"start_time\":\"47.5\",\"end_time\":\"48.18\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"always\"}],\"type\":\"pronunciation\"},{\"start_time\":\"48.18\",\"end_time\":\"48.6\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"have\"}],\"type\":\"pronunciation\"},{\"start_time\":\"48.61\",\"end_time\":\"49.0\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"to\"}],\"type\":\"pronunciation\"},{\"start_time\":\"49.01\",\"end_time\":\"49.41\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"be\"}],\"type\":\"pronunciation\"},{\"start_time\":\"49.62\",\"end_time\":\"50.36\",\"alternatives\":[{\"confidence\":\"0.6113\",\"content\":\"concealed\"}],\"type\":\"pronunciation\"},{\"start_time\":\"50.37\",\"end_time\":\"50.75\",\"alternatives\":[{\"confidence\":\"0.9826\",\"content\":\"Don't\"}],\"type\":\"pronunciation\"},{\"start_time\":\"50.75\",\"end_time\":\"51.26\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"feel\"}],\"type\":\"pronunciation\"},{\"start_time\":\"51.34\",\"end_time\":\"51.75\",\"alternatives\":[{\"confidence\":\"0.9873\",\"content\":\"don't\"}],\"type\":\"pronunciation\"},{\"start_time\":\"51.75\",\"end_time\":\"52.14\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"let\"}],\"type\":\"pronunciation\"},{\"start_time\":\"52.15\",\"end_time\":\"52.61\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"them\"}],\"type\":\"pronunciation\"},{\"start_time\":\"52.61\",\"end_time\":\"55.61\",\"alternatives\":[{\"confidence\":\"0.994\",\"content\":\"know\"}],\"type\":\"pronunciation\"},{\"start_time\":\"55.62\",\"end_time\":\"56.17\",\"alternatives\":[{\"confidence\":\"0.9905\",\"content\":\"Well\"}],\"type\":\"pronunciation\"},{\"alternatives\":[{\"confidence\":\"0.0\",\"content\":\",\"}],\"type\":\"punctuation\"},{\"start_time\":\"56.17\",\"end_time\":\"56.65\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"now\"}],\"type\":\"pronunciation\"},{\"start_time\":\"56.66\",\"end_time\":\"57.05\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"that\"}],\"type\":\"pronunciation\"},{\"start_time\":\"57.06\",\"end_time\":\"57.12\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"we\"}],\"type\":\"pronunciation\"},{\"start_time\":\"57.12\",\"end_time\":\"60.96\",\"alternatives\":[{\"confidence\":\"0.6819\",\"content\":\"Oh\"}],\"type\":\"pronunciation\"},{\"alternatives\":[{\"confidence\":\"0.0\",\"content\":\",\"}],\"type\":\"punctuation\"},{\"start_time\":\"60.97\",\"end_time\":\"61.26\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"let\"}],\"type\":\"pronunciation\"},{\"start_time\":\"61.26\",\"end_time\":\"61.43\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"it\"}],\"type\":\"pronunciation\"},{\"start_time\":\"61.44\",\"end_time\":\"62.52\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"go\"}],\"type\":\"pronunciation\"},{\"start_time\":\"62.53\",\"end_time\":\"63.34\",\"alternatives\":[{\"confidence\":\"0.9809\",\"content\":\"Can't\"}],\"type\":\"pronunciation\"},{\"start_time\":\"63.34\",\"end_time\":\"63.66\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"hold\"}],\"type\":\"pronunciation\"},{\"start_time\":\"63.66\",\"end_time\":\"63.78\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"it\"}],\"type\":\"pronunciation\"},{\"start_time\":\"63.78\",\"end_time\":\"64.26\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"back\"}],\"type\":\"pronunciation\"},{\"start_time\":\"64.26\",\"end_time\":\"66.16\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"anymore\"}],\"type\":\"pronunciation\"},{\"start_time\":\"66.24\",\"end_time\":\"66.52\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"Let\"}],\"type\":\"pronunciation\"},{\"start_time\":\"66.52\",\"end_time\":\"66.69\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"it\"}],\"type\":\"pronunciation\"},{\"start_time\":\"66.7\",\"end_time\":\"67.92\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"go\"}],\"type\":\"pronunciation\"},{\"start_time\":\"67.93\",\"end_time\":\"68.24\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"Let\"}],\"type\":\"pronunciation\"},{\"start_time\":\"68.24\",\"end_time\":\"68.4\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"it\"}],\"type\":\"pronunciation\"},{\"start_time\":\"68.41\",\"end_time\":\"69.57\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"go\"}],\"type\":\"pronunciation\"},{\"start_time\":\"69.58\",\"end_time\":\"70.05\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"Turn\"}],\"type\":\"pronunciation\"},{\"start_time\":\"70.05\",\"end_time\":\"70.64\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"away\"}],\"type\":\"pronunciation\"},{\"start_time\":\"70.64\",\"end_time\":\"70.9\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"and\"}],\"type\":\"pronunciation\"},{\"start_time\":\"70.91\",\"end_time\":\"71.61\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"slam\"}],\"type\":\"pronunciation\"},{\"start_time\":\"71.62\",\"end_time\":\"71.73\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"e\"}],\"type\":\"pronunciation\"},{\"start_time\":\"71.73\",\"end_time\":\"74.97\",\"alternatives\":[{\"confidence\":\"0.9743\",\"content\":\"o\"}],\"type\":\"pronunciation\"},{\"start_time\":\"74.98\",\"end_time\":\"76.17\",\"alternatives\":[{\"confidence\":\"0.7531\",\"content\":\"care\"}],\"type\":\"pronunciation\"},{\"start_time\":\"76.14\",\"end_time\":\"76.89\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"what\"}],\"type\":\"pronunciation\"},{\"start_time\":\"76.9\",\"end_time\":\"77.28\",\"alternatives\":[{\"confidence\":\"0.9845\",\"content\":\"they're\"}],\"type\":\"pronunciation\"},{\"start_time\":\"77.29\",\"end_time\":\"77.93\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"going\"}],\"type\":\"pronunciation\"},{\"start_time\":\"77.94\",\"end_time\":\"79.77\",\"alternatives\":[{\"confidence\":\"0.582\",\"content\":\"Thio\"}],\"type\":\"pronunciation\"},{\"start_time\":\"80.14\",\"end_time\":\"80.4\",\"alternatives\":[{\"confidence\":\"0.812\",\"content\":\"He\"}],\"type\":\"pronunciation\"},{\"start_time\":\"80.4\",\"end_time\":\"80.63\",\"alternatives\":[{\"confidence\":\"0.8684\",\"content\":\"let\"}],\"type\":\"pronunciation\"},{\"start_time\":\"80.63\",\"end_time\":\"80.74\",\"alternatives\":[{\"confidence\":\"0.9903\",\"content\":\"the\"}],\"type\":\"pronunciation\"},{\"start_time\":\"80.74\",\"end_time\":\"81.32\",\"alternatives\":[{\"confidence\":\"0.2724\",\"content\":\"store\"}],\"type\":\"pronunciation\"},{\"start_time\":\"81.33\",\"end_time\":\"83.75\",\"alternatives\":[{\"confidence\":\"0.5104\",\"content\":\"Ragen\"}],\"type\":\"pronunciation\"},{\"start_time\":\"84.12\",\"end_time\":\"84.82\",\"alternatives\":[{\"confidence\":\"0.9517\",\"content\":\"cold\"}],\"type\":\"pronunciation\"},{\"start_time\":\"84.82\",\"end_time\":\"85.18\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"Never\"}],\"type\":\"pronunciation\"},{\"start_time\":\"85.19\",\"end_time\":\"85.65\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"bothered\"}],\"type\":\"pronunciation\"},{\"start_time\":\"85.65\",\"end_time\":\"85.91\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"me\"}],\"type\":\"pronunciation\"},{\"start_time\":\"85.91\",\"end_time\":\"86.43\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"anyway\"}],\"type\":\"pronunciation\"},{\"alternatives\":[{\"confidence\":\"0.0\",\"content\":\".\"}],\"type\":\"punctuation\"},{\"start_time\":\"86.44\",\"end_time\":\"88.45\",\"alternatives\":[{\"confidence\":\"0.9576\",\"content\":\"Way\"}],\"type\":\"pronunciation\"},{\"start_time\":\"90.04\",\"end_time\":\"91.81\",\"alternatives\":[{\"confidence\":\"0.9812\",\"content\":\"It's\"}],\"type\":\"pronunciation\"},{\"start_time\":\"91.81\",\"end_time\":\"92.22\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"funny\"}],\"type\":\"pronunciation\"},{\"start_time\":\"92.22\",\"end_time\":\"92.38\",\"alternatives\":[{\"confidence\":\"0.9906\",\"content\":\"how\"}],\"type\":\"pronunciation\"},{\"start_time\":\"92.38\",\"end_time\":\"92.89\",\"alternatives\":[{\"confidence\":\"0.974\",\"content\":\"some\"}],\"type\":\"pronunciation\"},{\"start_time\":\"92.9\",\"end_time\":\"94.05\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"distance\"}],\"type\":\"pronunciation\"},{\"start_time\":\"94.26\",\"end_time\":\"94.92\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"makes\"}],\"type\":\"pronunciation\"},{\"start_time\":\"94.92\",\"end_time\":\"95.85\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"everything\"}],\"type\":\"pronunciation\"},{\"start_time\":\"95.85\",\"end_time\":\"96.33\",\"alternatives\":[{\"confidence\":\"0.6913\",\"content\":\"seem\"}],\"type\":\"pronunciation\"},{\"start_time\":\"96.33\",\"end_time\":\"98.37\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"smooth\"}],\"type\":\"pronunciation\"},{\"start_time\":\"98.38\",\"end_time\":\"98.91\",\"alternatives\":[{\"confidence\":\"0.991\",\"content\":\"Fears\"}],\"type\":\"pronunciation\"},{\"start_time\":\"98.91\",\"end_time\":\"99.11\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"and\"}],\"type\":\"pronunciation\"},{\"start_time\":\"99.11\",\"end_time\":\"99.52\",\"alternatives\":[{\"confidence\":\"0.6821\",\"content\":\"one's\"}],\"type\":\"pronunciation\"},{\"start_time\":\"99.52\",\"end_time\":\"100.58\",\"alternatives\":[{\"confidence\":\"0.9869\",\"content\":\"controlled\"}],\"type\":\"pronunciation\"},{\"start_time\":\"100.58\",\"end_time\":\"101.13\",\"alternatives\":[{\"confidence\":\"0.9968\",\"content\":\"me\"}],\"type\":\"pronunciation\"},{\"start_time\":\"101.14\",\"end_time\":\"101.81\",\"alternatives\":[{\"confidence\":\"0.6604\",\"content\":\"Can't\"}],\"type\":\"pronunciation\"},{\"start_time\":\"101.82\",\"end_time\":\"102.26\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"get\"}],\"type\":\"pronunciation\"},{\"start_time\":\"102.27\",\"end_time\":\"106.94\",\"alternatives\":[{\"confidence\":\"0.4484\",\"content\":\"Thio\"}],\"type\":\"pronunciation\"},{\"start_time\":\"106.95\",\"end_time\":\"107.28\",\"alternatives\":[{\"confidence\":\"0.9915\",\"content\":\"See\"}],\"type\":\"pronunciation\"},{\"start_time\":\"107.29\",\"end_time\":\"107.88\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"what\"}],\"type\":\"pronunciation\"},{\"start_time\":\"107.88\",\"end_time\":\"108.16\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"I\"}],\"type\":\"pronunciation\"},{\"start_time\":\"108.17\",\"end_time\":\"108.71\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"can\"}],\"type\":\"pronunciation\"},{\"start_time\":\"108.72\",\"end_time\":\"109.15\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"do\"}],\"type\":\"pronunciation\"},{\"start_time\":\"109.24\",\"end_time\":\"110.03\",\"alternatives\":[{\"confidence\":\"0.9923\",\"content\":\"Test\"}],\"type\":\"pronunciation\"},{\"start_time\":\"110.04\",\"end_time\":\"110.4\",\"alternatives\":[{\"confidence\":\"0.9996\",\"content\":\"the\"}],\"type\":\"pronunciation\"},{\"start_time\":\"110.41\",\"end_time\":\"110.65\",\"alternatives\":[{\"confidence\":\"0.682\",\"content\":\"list\"}],\"type\":\"pronunciation\"},{\"start_time\":\"110.64\",\"end_time\":\"110.95\",\"alternatives\":[{\"confidence\":\"0.3791\",\"content\":\"Mm\"}],\"type\":\"pronunciation\"},{\"start_time\":\"110.95\",\"end_time\":\"111.42\",\"alternatives\":[{\"confidence\":\"0.5242\",\"content\":\"It's\"}],\"type\":\"pronunciation\"},{\"start_time\":\"111.42\",\"end_time\":\"111.71\",\"alternatives\":[{\"confidence\":\"0.3274\",\"content\":\"and\"}],\"type\":\"pronunciation\"},{\"start_time\":\"112.38\",\"end_time\":\"112.72\",\"alternatives\":[{\"confidence\":\"0.3312\",\"content\":\"you\"}],\"type\":\"pronunciation\"},{\"start_time\":\"112.73\",\"end_time\":\"113.12\",\"alternatives\":[{\"confidence\":\"0.4466\",\"content\":\"know\"}],\"type\":\"pronunciation\"},{\"start_time\":\"113.13\",\"end_time\":\"113.37\",\"alternatives\":[{\"confidence\":\"0.4991\",\"content\":\"I\"}],\"type\":\"pronunciation\"},{\"start_time\":\"113.38\",\"end_time\":\"113.9\",\"alternatives\":[{\"confidence\":\"0.5142\",\"content\":\"know\"}],\"type\":\"pronunciation\"},{\"start_time\":\"113.91\",\"end_time\":\"114.32\",\"alternatives\":[{\"confidence\":\"0.5635\",\"content\":\"wrote\"}],\"type\":\"pronunciation\"},{\"start_time\":\"114.33\",\"end_time\":\"114.81\",\"alternatives\":[{\"confidence\":\"0.9798\",\"content\":\"no\"}],\"type\":\"pronunciation\"},{\"start_time\":\"114.82\",\"end_time\":\"115.26\",\"alternatives\":[{\"confidence\":\"0.8244\",\"content\":\"booze\"}],\"type\":\"pronunciation\"},{\"start_time\":\"115.26\",\"end_time\":\"115.39\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"for\"}],\"type\":\"pronunciation\"},{\"start_time\":\"115.39\",\"end_time\":\"116.88\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"me\"}],\"type\":\"pronunciation\"},{\"start_time\":\"116.89\",\"end_time\":\"119.03\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"e\"}],\"type\":\"pronunciation\"},{\"start_time\":\"119.03\",\"end_time\":\"127.95\",\"alternatives\":[{\"confidence\":\"0.9478\",\"content\":\"o\"}],\"type\":\"pronunciation\"},{\"start_time\":\"128.07\",\"end_time\":\"129.07\",\"alternatives\":[{\"confidence\":\"0.9995\",\"content\":\"oh\"}],\"type\":\"pronunciation\"},{\"alternatives\":[{\"confidence\":\"0.0\",\"content\":\",\"}],\"type\":\"punctuation\"},{\"start_time\":\"129.08\",\"end_time\":\"129.49\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"let\"}],\"type\":\"pronunciation\"},{\"start_time\":\"129.49\",\"end_time\":\"129.55\",\"alternatives\":[{\"confidence\":\"0.9356\",\"content\":\"me\"}],\"type\":\"pronunciation\"},{\"start_time\":\"129.55\",\"end_time\":\"131.17\",\"alternatives\":[{\"confidence\":\"0.9265\",\"content\":\"Oh\"}],\"type\":\"pronunciation\"},{\"alternatives\":[{\"confidence\":\"0.0\",\"content\":\",\"}],\"type\":\"punctuation\"},{\"start_time\":\"131.18\",\"end_time\":\"131.53\",\"alternatives\":[{\"confidence\":\"0.597\",\"content\":\"you\"}],\"type\":\"pronunciation\"},{\"start_time\":\"131.53\",\"end_time\":\"131.61\",\"alternatives\":[{\"confidence\":\"0.5604\",\"content\":\"know\"}],\"type\":\"pronunciation\"},{\"start_time\":\"152.99\",\"end_time\":\"153.31\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"my\"}],\"type\":\"pronunciation\"},{\"start_time\":\"153.32\",\"end_time\":\"154.06\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"power\"}],\"type\":\"pronunciation\"},{\"start_time\":\"154.07\",\"end_time\":\"155.12\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"flurries\"}],\"type\":\"pronunciation\"},{\"start_time\":\"155.12\",\"end_time\":\"155.58\",\"alternatives\":[{\"confidence\":\"0.9898\",\"content\":\"through\"}],\"type\":\"pronunciation\"},{\"start_time\":\"155.58\",\"end_time\":\"156.08\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"the\"}],\"type\":\"pronunciation\"},{\"start_time\":\"156.09\",\"end_time\":\"156.56\",\"alternatives\":[{\"confidence\":\"0.9933\",\"content\":\"air\"}],\"type\":\"pronunciation\"},{\"start_time\":\"156.56\",\"end_time\":\"157.31\",\"alternatives\":[{\"confidence\":\"0.9929\",\"content\":\"into\"}],\"type\":\"pronunciation\"},{\"start_time\":\"157.32\",\"end_time\":\"157.59\",\"alternatives\":[{\"confidence\":\"0.697\",\"content\":\"that\"}],\"type\":\"pronunciation\"},{\"start_time\":\"157.6\",\"end_time\":\"159.73\",\"alternatives\":[{\"confidence\":\"0.8937\",\"content\":\"around\"}],\"type\":\"pronunciation\"},{\"start_time\":\"159.74\",\"end_time\":\"160.29\",\"alternatives\":[{\"confidence\":\"0.9895\",\"content\":\"my\"}],\"type\":\"pronunciation\"},{\"start_time\":\"160.29\",\"end_time\":\"160.88\",\"alternatives\":[{\"confidence\":\"0.4762\",\"content\":\"sword\"}],\"type\":\"pronunciation\"},{\"start_time\":\"160.88\",\"end_time\":\"161.12\",\"alternatives\":[{\"confidence\":\"0.4219\",\"content\":\"They\"}],\"type\":\"pronunciation\"},{\"start_time\":\"161.13\",\"end_time\":\"162.63\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"spiraling\"}],\"type\":\"pronunciation\"},{\"start_time\":\"162.63\",\"end_time\":\"163.02\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"in\"}],\"type\":\"pronunciation\"},{\"start_time\":\"163.02\",\"end_time\":\"163.79\",\"alternatives\":[{\"confidence\":\"0.8633\",\"content\":\"frozen\"}],\"type\":\"pronunciation\"},{\"start_time\":\"163.8\",\"end_time\":\"170.96\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"froth\"}],\"type\":\"pronunciation\"},{\"start_time\":\"173.24\",\"end_time\":\"173.3\",\"alternatives\":[{\"confidence\":\"0.2489\",\"content\":\"on\"}],\"type\":\"pronunciation\"},{\"start_time\":\"173.31\",\"end_time\":\"174.21\",\"alternatives\":[{\"confidence\":\"0.727\",\"content\":\"the\"}],\"type\":\"pronunciation\"},{\"start_time\":\"174.29\",\"end_time\":\"175.92\",\"alternatives\":[{\"confidence\":\"0.9998\",\"content\":\"way\"}],\"type\":\"pronunciation\"},{\"start_time\":\"175.93\",\"end_time\":\"177.05\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"back\"}],\"type\":\"pronunciation\"},{\"start_time\":\"177.1\",\"end_time\":\"177.72\",\"alternatives\":[{\"confidence\":\"0.9969\",\"content\":\"Past\"}],\"type\":\"pronunciation\"},{\"start_time\":\"177.72\",\"end_time\":\"177.97\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"is\"}],\"type\":\"pronunciation\"},{\"start_time\":\"177.97\",\"end_time\":\"178.39\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"in\"}],\"type\":\"pronunciation\"},{\"start_time\":\"180.14\",\"end_time\":\"180.2\",\"alternatives\":[{\"confidence\":\"0.7742\",\"content\":\"the\"}],\"type\":\"pronunciation\"},{\"start_time\":\"180.2\",\"end_time\":\"193.76\",\"alternatives\":[{\"confidence\":\"0.3967\",\"content\":\"Oh\"}],\"type\":\"pronunciation\"},{\"alternatives\":[{\"confidence\":\"0.0\",\"content\":\",\"}],\"type\":\"punctuation\"},{\"start_time\":\"210.4\",\"end_time\":\"210.97\",\"alternatives\":[{\"confidence\":\"0.3655\",\"content\":\"Cole\"}],\"type\":\"pronunciation\"},{\"start_time\":\"210.97\",\"end_time\":\"211.38\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"never\"}],\"type\":\"pronunciation\"},{\"start_time\":\"211.39\",\"end_time\":\"211.85\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"bothered\"}],\"type\":\"pronunciation\"},{\"start_time\":\"211.85\",\"end_time\":\"212.14\",\"alternatives\":[{\"confidence\":\"1.0\",\"content\":\"me\"}],\"type\":\"pronunciation\"},{\"start_time\":\"212.14\",\"end_time\":\"213.16\",\"alternatives\":[{\"confidence\":\"0.9999\",\"content\":\"anyway\"}],\"type\":\"pronunciation\"}]},\"status\":\"COMPLETED\"}"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "RlDeylOatqOc"
      },
      "source": [
        "If we want to change the ``.json`` file either to ``.docx`` or ``.csv`` to generate the data frame"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "MtFiIRkWogAv",
        "outputId": "0f5867fa-5903-45fc-ed89-efde842919f3"
      },
      "source": [
        "#change .json to .docx\n",
        "import tscribe\n",
        "tscribe.write(\"asrOutput.json\")"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "LetItGo_Disney.docx written in 0.28 seconds.\n"
          ],
          "name": "stdout"
        },
        {
          "output_type": "display_data",
          "data": {
            "text/plain": [
              "<Figure size 432x288 with 0 Axes>"
            ]
          },
          "metadata": {
            "tags": []
          }
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "Oel-mtY4ogAw",
        "outputId": "8ad9a3d7-dd3e-492d-fb23-7e1531ecbe1e"
      },
      "source": [
        "#change .json to .csv\n",
        "tscribe.write(\"asrOutput.json\", format=\"csv\", save_as=\"LetItGo_Disney.csv\")"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "LetItGo_Disney.csv written in 0.03 seconds.\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "_dfFrceqt4Y3"
      },
      "source": [
        "Above described the basic steps to use the SageMaker to do AWS Transcribe and generate ``.json`` file at the end."
      ]
    }
  ]
}
