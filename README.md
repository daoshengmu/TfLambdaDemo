# TfLambdaDemo
TensorFlow AWS Lambda Demo

This project is inspired by [mikepm35/TfLambdaDemo](https://github.com/mikepm35/TfLambdaDemo), and its original article is posted [here](https://medium.com/@mike.p.moritz/running-tensorflow-on-aws-lambda-using-serverless-5acf20e00033).

## How to Setup
1. Install serverless npm package via

    `npm install -g serverless`
2. Setup a AWS serverless environment via

    `serverless create --template aws-python`.
3. In order to run TensorFlow, we need to assign a specific version (3.6.5) of Python in its virutal environment by typing

    `virtualenv --python=python3.6.5 tflambdademo` &&
    `source tflambdademo/bin/activate`

4. For this demo, we are going to pick TensorFlow version 1.13.1. We also can chooser other versions of TensorFlow. However, AWS Lambda has a limitation of *Unzipped size must be smaller than 262144000 bytes*.

      For my understanding. v.1.13.1 is a smaller size version. Newer versions of TensorFlow framework are usually larger ones, that might cause us can't upload files to the server. Choosing a version of TensorFlow by inserting the following command,
      
      `pip install tensorflow==1.13.1` && `pip freeze > requirements.txt`.

5. Add `nodeploy` Python packages into the `serverless.yml`, also add `exclude` for the packages which you don't want to be uploaded to the server.
    ```
    package:
      exclude:
        - node_modules/**
        - tflambdademo/**
    ```
   After that, your unzip file size will be dropped dramatically, and you should be able to upload this zip file to AWS and without *the unzip file size is too large to upload* problem.

6. Deploy your code via a serverless command. (*you must lauch your Docker app in advance.*)

    `serverless deploy -v`

## How to run
1. After deploying to AWS Lambda, you will get your server **id** as below,
    ```
    POST - https://<id>.execute-api.us-west-2.amazonaws.com/dev/upload
    POST - https://<id>.execute-api.us-west-2.amazonaws.com/dev/train
    POST - https://<id>.execute-api.us-west-2.amazonaws.com/dev/infer
    ```
2. Use the server **id** to communicate with your AWS Gateway APIs. First of all, we can start to upload our training data to AWS.

    `curl -X POST https://<id>.execute-api.us-east-1.amazonaws.com/dev/upload`

    And, it will response a folder prefix defined by the epoch. `{"epoch": "1556995xxx"}`

3. Ask AWS Lambda to traing our data from the folder.

    `curl -X POST https://<id>.execute-api.us-east-1.amazonaws.com/dev/train -d '{"epoch": "1556995xxx"}'`

4. In the final step, we will ask AWS Lambda to execute an infer based on our training result.

    ```
    curl -X POST https://<id>.execute-api.us-east-1.amazonaws.com/dev/infer -d '{"epoch": "1556995xxx", "input": {"age": ["34"], "workclass": ["Private"], "fnlwgt": ["357145"], "education": ["Bachelors"], "education_num": ["13"], "marital_status": ["Married-civ-spouse"], "occupation": ["Prof-specialty"], "relationship": ["Wife"], "race": ["White"], "gender": ["Female"], "capital_gain": ["0"], "capital_loss": ["0"], "hours_per_week": ["50"], "native_country": ["United-States"], "income_bracket": [">50K"]}}'
    ```

    The result will look like this
    ```
    [{"logits": [1.088104009628296], "logistic": [0.7480245232582092], "probabilities": [0.25197547674179077, 0.7480245232582092], "class_ids": [1], "classes": ["1"]}]
    ```