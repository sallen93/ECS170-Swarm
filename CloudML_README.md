# Google CloudML

This README will show you the steps for getting your system set up to use our 
Google CloudML project for development and testing purposes.


## Getting Started

### Prerequisites

You will need to first download the Google Cloud SDK.
[Google Cloud SDK - 64 bit](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-201.0.0-linux-x86_64.tar.gz) - 19.7MB
[Google Cloud SDK - 32 bit](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-201.0.0-linux-x86.tar.gz) - 19.3MB


### Installing

Extract the archive into your prefered file location.

Once you have extracted the file you will need to run the install script. Open up a terminal in the folder you extracted the file to by either right-clicking inside the folder and selecting "Open in Terminal"or using the cd command to navigate to it. The run the following command.

```
./google-cloud-sdk/install.sh
```

## Initialize the SDK

Now that you have installed the SDK, you will need to use the gcloud command to perform a few SDK setup tasks.

To start setup, run the following.
```
gcloud init
```

It should then ask you to login. Select 'Y'
```
To continue, you must log in. Would you like to log in (Y/n)? Y
```

**Warning:** This stage may give errors as the additional user access is untested. If you have any issues, contact Sean. It probably isn't your fault.

Login to the @ucdavis.edu emails you provided to the team and click **Allow**.
**Note:** I have set your accounts to have full owner permissions. Please do not make any changes to the project settings without consulting the team first.

You will then see something like the following.
```
Pick cloud project to use: 
 [1] ecs170-swarmai-id
 [2] other-project-1
 [3] other-project-2
 [4] Create a new project
Please enter numeric choice or text value (must exactly match list
```

Select whichever number corresponds to "ecs170-swarmai-id"

It will then ask you
```
Do you want to configure a default Compute Region and Zone? (Y/n)?
```

Select Y and choose **us-west1-b**. You may need to use different regions later, but this should be fine for now. 

# Testing the ml-engine (optional)

Here we are going to do two things. First, we will test run an example model locally on your system. After that we will run it in the cloud.


## Installing TensorFlow and ml-engine sample
You can install TensorFlow natively by running the following command.
```
pip install --upgrade tensorflow
```
If this doesn't work, there is a very useful Getting Started page [here](https://www.tensorflow.org/install/install_linux#InstallingNativePip)

Once you have TensorFlow installed, test it to be sure everything works using the following commands.

Start a Python shell
```
python
```

Import TensorFlow
```
import tensorflow as tf
```

Create a string constant
```
hello = tf.constant('Hello, TensorFlow!')
```

Create a TensorFlow session
```
sess = tf.Session()
```
**Note:** You may get an error saying that your CPU supports additional instructions. This is not a fatal error and can be ignored. It is just saying TensorFlow can be configured to run more efficiently on your computer, but it is not important for this test.

Print the string back out.
```
print(sess.run(hello))
```

If you see the following you were successful!
```
Hello, TensorFlow!
```

### Download the ml engine sample
You can find the sample download [here](https://github.com/GoogleCloudPlatform/cloudml-samples/archive/master.zip)

Extract the files and navigate to the estimator directory
```
cd cloudml-samples-master/census/estimator
```

### Get the training data 
These command will download the data into a fancy new data folder.
```
mkdir data
gsutil -m cp gs://cloud-samples-data/ml-engine/census/data/* data/
```

You will also need the following variables to your local file paths.
```
TRAIN_DATA=$(pwd)/data/adult.data.csv
EVAL_DATA=$(pwd)/data/adult.test.csv
```

### Install the dependencies
```
pip install -r ../requirements.txt
```

## Running the trainer locally
Now that everything is downloaded, we'll go ahead and run it on your computer.

Set the output directory.
```
MODEL_DIR=output
```
If you are running locally multiple times, remove the old output before running again.
**SUPER WARNING:** If you have not set the previous variables before running this command you WILL destroy your OS. Don't do what I did...
```
rm -rf $MODEL_DIR/*
```

Now you can run the trainer with the following.
```
gcloud ml-engine local train \
    --module-name trainer.task \
    --package-path trainer/ \
    --job-dir $MODEL_DIR \
    -- \
    --train-files $TRAIN_DATA \
    --eval-files $EVAL_DATA \
    --train-steps 1000 \
    --eval-steps 100

```

### Launch TensorBoard
Tensorboard is a visualization tool that will allow you to see the metrics of the training.
You can launch TensorBoard with the following command.
```
tensorboard --logdir=$MODEL_DIR
```

Then open it in your web browser of choice with the URL http://localhost:6006

## Running the trainer on the cloud server
First we neeed to set up a Cloud Storage bucket. 
The bucket names need to be unique, so do something like "ecs170-swarmai-id-YOUR_NAME"
```
BUCKET_NAME=ecs170-swarmai-id-YOUR_NAME
```

Check the bucket name.
```
echo $BUCKET_NAME
```

Choose a region. The list of additional regions can be found [here](https://cloud.google.com/ml-engine/docs/tensorflow/regions).
```
REGION=us-central1
```

```
gsutil mb -l $REGION gs://$BUCKET_NAME
```

Now upload the data files to the Cloud Storage bucket
```
gsutil cp -r data gs://$BUCKET_NAME/data
```

Set the variables
```
TRAIN_DATA=gs://$BUCKET_NAME/data/adult.data.csv
EVAL_DATA=gs://$BUCKET_NAME/data/adult.test.csv
```

Move the JSON test file to the bucket
```
gsutil cp ../test.json gs://$BUCKET_NAME/data/test.json
```

Set the TEST_JSON variable.
```
TEST_JSON=gs://$BUCKET_NAME/data/test.json
```

### Run a single-instance trainer in the cloud
Now that the bucket is set up we can run it in the cloud.

Select a unique job name. Again, it needs to be unique (even against previous iterations) so go with something like "census_single_YOUR-NAME_1"
```
JOB_NAME=census_single_YOUR-NAME_1
```

Specify the output directory.
```
OUTPUT_PATH=gs://$BUCKET_NAME/$JOB_NAME
```

Run the training instance.
```
gcloud ml-engine jobs submit training $JOB_NAME \
    --job-dir $OUTPUT_PATH \
    --runtime-version 1.4 \
    --module-name trainer.task \
    --package-path trainer/ \
    --region $REGION \
    -- \
    --train-files $TRAIN_DATA \
    --eval-files $EVAL_DATA \
    --train-steps 1000 \
    --eval-steps 100 \
    --verbosity DEBUG
```

You can monitor the progress on the command-line or by going to **ML Engine** > **Jobs** on Google Cloud Platform Console

Once the process is complete you can view the results through TensorBoard again.
```
tensorboard --logdir=$OUTPUT_PATH
```

Then go to http://localhost:6006 in your browser.

The cost of the different processing options can be found [here](https://cloud.google.com/products/calculator/?authuser=1).

### Run distributed training in the cloud

Set the job name again.
```
JOB_NAME=census_dist_YOUR-NAME_1
```

Set the output path.
```
OUTPUT_PATH=gs://$BUCKET_NAME/$JOB_NAME
```

Run the trainer.
```
gcloud ml-engine jobs submit training $JOB_NAME \
    --job-dir $OUTPUT_PATH \
    --runtime-version 1.4 \
    --module-name trainer.task \
    --package-path trainer/ \
    --region $REGION \
    --scale-tier STANDARD_1 \
    -- \
    --train-files $TRAIN_DATA \
    --eval-files $EVAL_DATA \
    --train-steps 1000 \
    --verbosity DEBUG  \
    --eval-steps 100
```

You can view the logs the same way you did with single-instance training.

### Hyperparameter Tuning
Hyperparameter tuning for this example can be done through configuring the YAML file named hptuning_config.yaml.

Create the new job name and specify the config file.
```
HPTUNING_CONFIG=../hptuning_config.yaml
JOB_NAME=census_core_hptune_1
TRAIN_DATA=gs://$BUCKET_NAME/data/adult.data.csv
EVAL_DATA=gs://$BUCKET_NAME/data/adult.test.csv
```

Specify the output path.
```
OUTPUT_PATH=gs://$BUCKET_NAME/$JOB_NAME
```

Run the trainer.
```
gcloud ml-engine jobs submit training $JOB_NAME \
    --stream-logs \
    --job-dir $OUTPUT_PATH \
    --runtime-version 1.4 \
    --config $HPTUNING_CONFIG \
    --module-name trainer.task \
    --package-path trainer/ \
    --region $REGION \
    --scale-tier STANDARD_1 \
    -- \
    --train-files $TRAIN_DATA \
    --eval-files $EVAL_DATA \
    --train-steps 1000 \
    --verbosity DEBUG  \
    --eval-steps 100
```
