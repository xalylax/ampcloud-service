# AmpCloud Service

Welcome to AmpLabs! AmpLabs makes working with battery data easy, 

With our platforms we enable battery Observability.
Battery Observability tells your team critical information about your energy storage assets such as if, when, and how your battery is performing and how it compares to industrial models and forecasts.

## Requirements
To ensure a seamless development and setup experience, it is recommended to install the following essential components on your computer prior to starting the setup process:

* Python
* Node (React.js)
* Docker
* AWS CLI
* Serverless Framework
* Clone the following repos - UI and server 

Please ensure that you have the necessary dependencies installed and ready to use on your system. To confirm their functionality, you can execute the following commands:

```bash
node --version
```
```bash
python3 --version
```
```bash
aws --version
```

If it doesn't return any errors, you are good to go.

# AWS System Diagram
![SD2-a3a1cf97d6f53e3355113ccc5e59a277](https://github.com/amplabs-ai/ampcloud-service/assets/139910009/3af3e2fe-26c3-46d0-bcf5-e9870bb5d899)


# Deploy Our Application
AmpLabs makes **working with battery** data easy

We help you to analyze your battery against industry standard models and forecasts.

## Build your site for production:

### Ejecting and Building UI
To deploy we have to build the UI and attach that to the server and then we can deploy the server only.

1. **Creating env file used in UI**

Make a file called `.env`
inside the `amplabs-ui\.env`

Copy the content provided below inside the env and add your credentials related to AWS there

```
REACT_APP_ENV="production"
REACT_APP_PROD_URI="https://app.amplabs.ai/"
REACT_APP_PK_KEY=<Key>
REACT_APP_PROD_AUTH0_DOMAIN=<Auth0 Domain>
REACT_APP_PROD_AUTH0_CLIENT_ID=<Auth0 ClientID>
HTTPS=true
REACT_APP_AWS_ACCESS_KEY_ID= <AWS Access key>
REACT_APP_AWS_SECRET_ACCESS_KEY= <AWS Secret key>
REACT_APP_DEV_UPLOAD_S3_BUCKET= <Bucket name>
REACT_APP_PROD_UPLOAD_S3_BUCKET= <Bucket name>
REACT_APP_AWS_REGION= <Region of AWS>
```

2. **Steps to build the UI**
    1. Add the following code under scripts in package.json
    ```
    "eject": "react-scripts eject"
    ```
    2. Run the following in command line
    ```
    npm install 
    ```
    3. Run the following in command line
    ```
    npm run eject
    ```
    4. Change following in `amplabs-ui/config/path.js` file:
    ```
    appBuild: resolveApp('../../static/react')
    ```
    5. Remove `static/` everywhere from `amplabs-ui/config/webpack.config.js` file.
    6. Open the `amplabs-ui/config/webpack.config.js` file:
    * Goto HtmlWebpackPlugin function and add the following after the template line:
    ```
    filename: "../../templates/index.html",  
    ```
    7. Add the following in `amplabs-ui\public\index.html`
    ```
    <script> window.token="{{flask_token}}"</script>
    ```
    8. Add the following in amplabs-ui\src\App.js after "return(" statement in next line:
    ```
    <p>My Token = {window.token}</p>
    ```
    9. Add the following in amplabs-ui package.json file before dependencies:
    ```
    "homepage": "/static/react",
     ```
    10. Now run the command in command line -
    ```
    npm run build
    ```

This build will create two folders named `static` and `templates` in the root directory.

### Attaching build to server -
There must be two folders created named `static` and `templates` in the root directory of UI

Copy those folders and paste them in `app` folder of server

### Preparing server for deployments -

You need to set serverless framework. For setting up serverless you can look to serverless framework [Docs](https://www.serverless.com/framework/docs/getting-started).

After installing add the following files in the root directory of server

1. **Serverless.yml**
Make a file named **serverless.yml** inside `ampcloud-service\serverless.yml`
```
service: <your service name>

provider:
name: <service provider name Ex:AWS>
ecr:
    images:
        appimage:
            path: ./

functions:
app:
    image:
     name: <your image name>
timeout: <set the timeout>
events:
  - http: ANY /
  - http: "ANY /{proxy+}"
```
2. **Handler.py**
Make a file named **handler.py** inside `ampcloud-service\handler.py`
```
import serverless_wsgi
from app.server import app


def handler(event, context):
   return serverless_wsgi.handle_request(app, event, context)
```
3. **Dockerfile**
Make a **Dockerfile** using given template inside `ampcloud-service\Dockerfile`
```
FROM public.ecr.aws/lambda/python:3.7
COPY . ${LAMBDA_TASK_ROOT}
COPY requirements.txt  .
RUN pip3 install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"
CMD [ "handler.handler" ]
```
4. **Dockerignore**
Make a **dockerignore** using given template inside `ampcloud-service\.dockerignore`
```
#.dockerignore
__pycache__/
.git/
.serverless/
.gitignore
.dockerignore
serverless.yml
```
5. **Creating env file used in backend**
Make a file called **.env** inside the `app\.env`

Copy the content provided below inside the **.env** file and add your credentials related to AWS there
```
 ENV=production
 AMPLABS_PROD_DB_URL=<url of redshift database>
 AUTH0_PROD_DOMAIN=<Prod Domain>
 AUTH0_AUDIENCE=https://amplabs.server
 AWS_ACCESS_KEY_ID_1 =<AWS Access key>
 AWS_SECRET_ACCESS_KEY_1 =<AWS Secret key>
 TEMPLATE_DOMAIN_NAME_SDB =<domain name for template SDB>
 TRI_BUCKET_NAME =<s3 bucket name>
 AWS_IAM_ROLE =<arn to AWS user role>
 AWS_REGION =<Region of AWS>
 S3_DATA_PROD_BUCKET =<s3 bucket name>
 REDSHIFT_DATABASE =<redshift database name>
 REDSHIFT_SCHEMA =<redshift schema type name>
 STATUS_DOMAIN_NAME_SDB =<domain name for status SDB>
```
After adding these files run `sls deploy` in the server directory to deploy the application.

> **Warning** 
> Before executing sls deploy make sure to configure your aws services. Refer to [Aws Services Used](http://amplabs-docs.s3-website-us-east-1.amazonaws.com/docs/tutorial-basics/AWS%20Services%20Used) doc.

# AWS clour deployment flow chart
![flow_chart-edae1ce6ce9cf60e80b99515cbd9b886](https://github.com/amplabs-ai/ampcloud-service/assets/139910009/01ec1ce8-1d23-437d-bb06-879b5d02de69)






# AWS Services Used

We mainly used 3 services for our Infrastructure

1. **S3 buckets**
Used to temporarily store responses from API

In S3 create a bucket. Inside it create 4 sub folders namely

*`raw`: all the file upload are saved here
*`tri`: used for storing tri data
*`sample`: all the tutorial related data are stored here
*`response`: all the plots response are saved here

2. **Redshift as database**
We are using Redshift as database to store the content of files that are uploaded on production. Following are the tables and there schema

* cell_metadata
* Cycle_metadata
* shared_dashboard
* user_plan
* cycle_timeseries
* cycle_stats
For more detailed description about how our database is designed you can refer to [Database Design](http://amplabs-docs.s3-website-us-east-1.amazonaws.com/docs/tutorial-basics/Database%20Design) Doc

3. **SimpleDB**
It is used to store template as well as status call for upload details

4. **Load balancer**
We have a load balancer in AWS in which we have a HTTPS listener which forwards to the target group assigned to it.






# Database Design
## Redshift as database

We are using Redshift as database to store the content of files that are uploaded on production. Following are the tables and there schema

*  **cell_metadata**
![cell_metadata-9761173f991112f4c3a6de4c2cb96638](https://github.com/amplabs-ai/ampcloud-service/assets/139910009/0b462d1f-55bf-4255-a3b8-c4cf7f4019a9)
*  **cell_metadata**
![cycle_metadata-e373909ea7f318b6a534b611b23d8c88](https://github.com/amplabs-ai/ampcloud-service/assets/139910009/536d6378-7483-49c7-bcfe-7d3197a3c565)
* **shared_dashboard**
![shared_dashboard-669bb2ec3f89d06f914b8cf407149837](https://github.com/amplabs-ai/ampcloud-service/assets/139910009/c0bea5dc-e2e3-46e2-af75-f1c2131b3704)
* **user_plan**
![user_plan-40a0c5f676372b655c2a1894bc72bc6f](https://github.com/amplabs-ai/ampcloud-service/assets/139910009/9c4880ce-fb0b-420c-91f2-3af78750eeab)
* **cycle_timeseries**
![cycle_timeseries-ec945c10e05690aff34659b2349c726d](https://github.com/amplabs-ai/ampcloud-service/assets/139910009/f877e4da-0b2c-4525-b0fd-5199db7cbc13)
* **cycle_stats**
![cycle_stats-be66089ae9963795fda81a9edac4d7c1](https://github.com/amplabs-ai/ampcloud-service/assets/139910009/efcd4c4e-3cb5-4dec-bef6-6fdb8bf8d238)






# Conceptual Schematic Diagram

AmpLabs Cloud Service realizes this conceptual idea

![schematic](assets/images/schematic.png)

# How to Contribute:

## Suggest Improvements
1. Report bugs, feature or change requests by clicking 'Issues' on top bar
2. Fill out form documenting improvement you would like to see.
3. Label appropriately
4. Assign Issue to Milestone (optional)
5. Assign Issue to Project (optional)

## Develop Code or Write Documentation

1. Fork Repository by clicking 'Fork' on top right of this page.
2. Create branches to develop new code.
3. When complete submit a 'Pull Request' from repository and assign to a Maintainer for review
