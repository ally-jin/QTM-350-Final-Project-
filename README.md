# QTM-350-Final-Project-
## Amazon Transcribe 
Amazon Transcribe is a AWS ML service that converts audio speech to text. Our project looks into the accurancy of the transcribe service.
To test the accurancy of Amazon Transcribe, we used songs given different speeds, levels of background noise, and artists' pronunciations. Then, we compared the lyrics transribed by Amazon Transcribed to the actual lyrics. 

## Option 1: Use the AWS Transcribe from the SageMaker

Please refer to the notebook in this [repository](https://github.com/ally-jin/QTM-350-Final-Project-.git) called QTM350_FinalProject_AWSTranscribe_fromSageMaker.ipynb.

In general, we will be using following code to automatically transcribe our song into a json file. 

```
#start transcription 
transcribe.start_transcription_job(
    TranscriptionJobName= "sample_name",
    Media={'MediaFileUri': 's3://sample_name.mp3'},
    MediaFormat='mp3',
    LanguageCode='en-US'
)
```

## Option 2: Create an Audio Transcript with Amazon Transcribe

### Step 1. Create a S3 bucket and upload sample audio file
You will download a sample audio file, create a S3 bucket, then upload the sample mp3 file to the S3 bucket. Transcribe accesses audio and video files for transcription exclusively from S3 buckets.

a.  Search **S3** in the AWS services search bar and select **S3** to open the console

b.  In the S3 dashboard choose **Create bucket**.

c.  Enter a unique bucket name. Bucket names must be unique across all existing bucket names in Amazon S3. There are a number of other restrictions on S3 bucket names as well. Then select a **Region** to create your bucket in.

d.  You have many useful options for your S3 bucket including Versioning, Server Access Logging, Tags, Object-level Logging and Default Encryption. We won't enable these features for this tutorial.

e.  In this step you have the ability to adjust permission settings for your S3 bucket during the S3 bucket creation process.
Leave the default values and select **Next**.

f. Review your configuration settings and select **Create bucket**. 

g. You will see your new bucket in the S3 console. Click on your bucket’s name to navigate to the bucket. Your bucket name will not be the same as pictured in the screenshot to the right.

h.  You are in your bucket’s home page. Select **Upload**.

i.  Upload your mp3 file by selecting **Add files** and selecting the file OR dragging the mp3 file to the upload box. Select **Upload**.

j.  Select the checkbox next to the mp3 file name in your bucket. A file detail pane will be displayed for the mp3 file name file. Copy the link to the file and save it for use later in the tutorial.

#### Step 1: Our project 
In our project, for each specific song, we uploaded mp3 files of slow(x0.5), original (1.0x), and fast (x2.0) speeds into our project bucket. 
  

### Step 2. Create a Transcription Job

a.  From the top menu bar, select **Services** then begin typing *Transcribe* in the search bar and select **Amazon Transcribe** to open the service console.

b.  On the **Amazon Transcribe** console main page, open the navigation pane and click **Transcription jobs**.

c.  On the Transcription jobs page, click **Create job**.

d.  On the Create transcription job page, in the Name field, type *sample-transcription-job*.
 
   Leave the default Language as English.
   
   In the Input file location on S3 field, paste the link to the sample file in your S3 bucket. Leave the default Format of mp3.

e. All the other options will be put on default. We did not use any of these options.

f. Select **Create** to start your transcription job.  

#### Step 2. Our project

We created a transcription job for all the songs of all speeds in our data. 

### Step 3. Review Transcription Job

a. After you click the **Create** button, you will be taken to the **Transcription jobs** screen. It will show the status of *sample-transcription-job*. The status can be In progress, Complete, or Failed.

b. When the status is **Complete**, click on the *sample-transcription-job* link in the **Name** column to view the transcription results.

c. Next you will see the *sample-transcription-job* details. Scroll down to the **Transcription** panel to view the transcription job output. In the JSON pane you can view the transcription results as it would be returned from the Transcribe API or AWS CLI. 


*For more information on these three steps, please refer to **Resources** : [Audio Transcript using Amazon Transcribe](https://aws.amazon.com/getting-started/hands-on/create-audio-transcript-transcribe/#)*

#### Step 3. Our project

The transcription job in the .json format was put back into our bucket. We turned the songs transcription results into a data frame in order to check for the accuracy. 

#### Step 4. Our project: Accuracy 
To check for the accuracy, we used the **JiWER** python package to approximate the Word Error Rate (WER). 

a. We installed the **JiWER** package and imported **wer**

```
$ pip install jiwer

from jiwer import wer

ground_truth = "this represents the correct song lyrics"

hypothesis = "this represents the transcribed song lyrics"

error = wer(
ground_truth, hypothesis)

```
We gathered all the word error rate for each song and its different speeds and put them into our data frame. 

#### Step 5. Our project: Dataframe

We put our songs, speed, genre, and word error rate into our data frame. We exported it as a csv. This was our data for the analysis portion of our project. 

# Architecture Overview 
![architecture](https://github.com/ally-jin/QTM-350-Final-Project-/blob/main/Transcribe-Page-2.png)

We initally downloaded all of our song files into mp3. We changed the speed of each song to either 0.5x or 2.0x and also downloaded it as mp3 files. Using [S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/Introduction.html), we uploaded all of our mp3 files into our project S3 bucket. Then, we used the [Amazon Transcribe](https://docs.aws.amazon.com/transcribe/index.html) service to get a transcription of each song and speed into json file. These transcription json files were put back into our project S3 bucket, so we could check the accuracy of each song and its speeds with the JiWER python package. Lastly, we created a dataframe showing speed, genre, and accuracy. This dataframe is turned into a csv file to be used for our analysis. 

## The solutions uses the following resources:

1. [Audio Transcript using Amazon Transcribe](https://aws.amazon.com/getting-started/hands-on/create-audio-transcript-transcribe/#)

2. [JiWER Python Package](https://pypi.org/project/jiwer/)

3. [Amazon Transcribe Documentation](https://docs.aws.amazon.com/transcribe/index.html)

4. [Amazon S3 Documentation](https://docs.aws.amazon.com/AmazonS3/latest/dev/Introduction.html)


