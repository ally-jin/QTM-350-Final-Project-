# QTM-350-Final-Project-
## Amazon Transcribe 
Amazon Transcribe is a AWS ML service that converts audio speech to text. Our project looks into the accurancy of the transcribe service.
To test the accurancy of Amazon Transcribe, we used songs given different speeds, levels of background noise, and artists' pronunciations. Then, we compared the lyrics transribed by Amazon Transcribed to the actual lyrics. 

## Option 1: Create an Audio Transcript with Amazon Transcribe

### Step 1. Create a S3 bucket and upload sample audio file
You will download a sample audio file, create a S3 bucket, then upload the sample mp3 file to the S3 bucket. Transcribe accesses audio and video files for transcription exclusively from S3 buckets.

a.  Search **S3** in the AWS services search bar and select **S3** to open the console
b.  In the S3 dashboard choose Create bucket.
c.  Enter a unique bucket name. Bucket names must be unique across all existing bucket names in Amazon S3. There are a number of other restrictions on S3 bucket names as well. Then select a Region to create your bucket in.
d.  You have many useful options for your S3 bucket including Versioning, Server Access Logging, Tags, Object-level Logging and Default Encryption. We won't enable these features for this tutorial.
e.  In this step you have the ability to adjust permission settings for your S3 bucket during the S3 bucket creation process.
Leave the default values and select **Next**.
f. Review your configuration settings and select **Create bucket**. 
g. You will see your new bucket in the S3 console. Click on your bucket’s name to navigate to the bucket. Your bucket name will not be the same as pictured in the screenshot to the right.
h.  You are in your bucket’s home page. Select **Upload**.
i.  Upload your mp3 file by selecting **Add files** and selecting the file OR dragging the mp3 file to the upload box. Select **Upload**.

### Step 2. Create a Transcription Job
a.  From the top menu bar, select **Services** then begin typing *Transcribe* in the search bar and select **Amazon Transcribe** to open the service console.
b.  On the **Amazon Transcribe** console main page, open the navigation pane and click **Transcription jobs**.

c.  On the Transcription jobs page, click **Create job**.
d.  On the Create transcription job page, in the Name field, type *sample-transcription-job*.
 
   Leave the default Language as English.
   In the Input file location on S3 field, paste the link to the sample file in your S3 bucket. Leave the default Format of mp3.

e. All the other options will be put on default. We did not use any of these options.
f. Select **Create** to start your transcription job.  

### Step 3. Review Transcription Job
a. After you click the **Create** button, you will be taken to the **Transcription jobs** screen. It will show the status of *sample-transcription-job*. The status can be In progress, Complete, or Failed.
b. When the status is **Complete**, click on the *sample-transcription-job* link in the **Name** column to view the transcription results.
c. Next you will see the *sample-transcription-job* details. Scroll down to the **Transcription** panel to view the transcription job output. In the JSON pane you can view the transcription results as it would be returned from the Transcribe API or AWS CLI. 

 
.json and turned it back into word back into the bucket cvs word document to check for accuracy

