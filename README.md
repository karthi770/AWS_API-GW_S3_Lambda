## Serverless Usecase - 2 - API GW, Lambda and S3:
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/87fe0093-4578-489c-85c7-95c65f6422f1)
### Create two S3 buckets:

![image](https://github.com/karthi770/Tetris_Game/assets/102706119/bd6cad47-5f00-4e38-bead-93116c75796c)

![image](https://github.com/karthi770/Tetris_Game/assets/102706119/db6cbb13-6480-4abb-b65b-d7d4b4dcb446)

![image](https://github.com/karthi770/Tetris_Game/assets/102706119/e8818f6f-bf68-4e2c-b9fc-c247535c0fb1)
Upload these json on the respective buckets.
```json
{
	"CustomerID": "01",
	"Product": "Coffee",
	"Address": "135, Rose avenue",
	"Quantity": 4
}
```
```json
{
	"CustomerID": "02",
	"Product": "Tea",
	"Address": "2015, Davenport",
	"Quantity": 4
}
```

### Part - 1
### Lambda Function

- Create a function using python runtime
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/42d75733-2164-4ca8-bfd1-bd4c443357ca)

- Increase the time out click on configuration tab → General configuration → increase the time
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/159bae14-4c5c-47a3-a6aa-09bf5696e838)

- Attach the policy to lambda role that can access the s3 bucket. Click on configuration tab → select permissions → click on the default role that has been created → attach the policy.
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/5f7782dc-ae3c-483d-965f-9c45b8e9d0ef)

![image](https://github.com/karthi770/Tetris_Game/assets/102706119/05841247-61c6-43a2-8f13-3ff3935fe10c)
>[!note]
>Difference between json loads and dumps

```python
import boto3
import json
client = boto3.client('s3')
def lambda_handler(event, context):
    resopnse = client.get_object(
        Bucket = 'first-json-bucket-01',
        Key = 'bucket01.json',
)

# convert from streaming to bytes
    data_bytes = resopnse['Body'].read()
# bytes to string
    data_strings = data_bytes.decode("UTF-8")
# convert strings to dictionary
    data_dict = json.loads(data_strings)
    return {
        'statusCode': 200,
        'body': data_dict
    }
    
```

- Result of the above code
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/34261846-a685-49b2-a5d5-3db92154349c)

### Invoke Lambda using API gateway
- Come back to the lambda function and add trigger to it. Select the trigger as API gateway → Create new API → REST API → Security IAM.
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/589d7b4a-c04a-4202-8794-7ffa0f3712ab)
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/574f357b-6b93-47cd-af67-e1b9c0594142)

- Click on the API gateway and create the resource
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/1571ccfa-6492-427d-aad2-c969cc0d4892)

- Create a Method and select the GET method → select the lambda function integration → select the created lambda function.
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/29efcc50-9efc-4162-86c4-5685dd39633c)
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/348013e2-f183-47f5-8280-8a69ab710315)

- Add a stage named DEV and then navigate to the GET method of the resource that we created.
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/fe878887-e081-4000-8894-f79efd5fff69)
- When we open the invoke URL we shall get the following result:
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/f27fb32f-0f23-4b7d-afb6-b2a2f59d260f)

### Part - 2 

#### Changes in the lambda codes
```python
import boto3
import json
client = boto3.client('s3')
def lambda_handler(event, context):
    bucket = event['bucket']
    key=event['key']

    resopnse = client.get_object(
        Bucket = bucket,
        Key = key,
)
# convert from streaming to bytes
    data_bytes = resopnse['Body'].read()
# bytes to string
    data_strings = data_bytes.decode("UTF-8")
# convert strings to dictionary
    data_dict = json.loads(data_strings)
    return {
        'statusCode': 200,
        'body': data_dict
    }
```
>[!note]
>We are trying to capture the bucket names from the event that we are creating. Notice the changes made after the` lambda_handler().` We are parameterizing the values of the bucket and the key values.

![image](https://github.com/karthi770/Tetris_Game/assets/102706119/60f123a1-7ca5-4996-b25f-55304d25c679)

![image](https://github.com/karthi770/Tetris_Game/assets/102706119/eb259ecf-b26b-4eb3-9978-649faee35428)

#### Changes in the API gateway

- Edit the method request → request validator → change it to the highlighted option.
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/0a3ff639-85f9-42e8-9922-c8dc54316984)
- Edit the URL query string parameter
![ezgif-5-4e7437337b](https://github.com/karthi770/Tetris_Game/assets/102706119/6ce04786-eafd-4f88-819f-3d873efec142)

- Edit the integration request
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/383d3e1e-58db-4cc0-bcf7-6bb1741d6aef)

- Make sure the request body passthrough has the recommended option:
  ![image](https://github.com/karthi770/Tetris_Game/assets/102706119/229f5c34-7f19-47c6-afd5-32cc48fed16e)

- Mapping template will ensure to grab the input the user is giving and send it to the event of the lambda function which will in turn route to the bucket.
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/dbe1bef9-fc9a-4103-851c-8130411b8d20)

```json
{
"bucket" : "$input.params("bucket")",
"key" : "$input.params("key")"
}
```

- Test the API to see if it gives the right result, pass the query strings:
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/0ce3b646-ab74-43ff-839e-4263f0f94ef9)
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/ee6908ac-7ebb-44f9-b240-a43b3e1336d7)
_We got the required response_

- Now deploy the API in Production stage
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/49b3dad4-255f-4a6a-b3ed-8c4d70f14df1)
_Once we deploy we get the invoke URL and that should be modified by giving the bucket name and key to get the result, see the URL below, *?bucket=first-json-bucket-02&key=bucket02.json* has been added extra at the end of the URL_
https://eiq3vi87j3.execute-api.us-east-1.amazonaws.com/Prod/apigateway_lambda/qty_drinks?bucket=first-json-bucket-02&key=bucket02.json
- Now we know that the API call is working.
![image](https://github.com/karthi770/Tetris_Game/assets/102706119/1e2e3e08-a598-42c2-babb-8e11afa82982)


