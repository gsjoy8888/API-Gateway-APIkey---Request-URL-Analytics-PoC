# API-Gateway-APIkey---Request-URL-Analytics-PoC

API Gateway APIkey - Request URL Analytics PoC

1. Create a S3 bucket named for example “api-gateway-logs-bucket”
2. Create a Kinesis Firehose named “amazon-apigateway-logs” and set the destination to S3 bucket “api-gateway-logs-bucket”. *Please be noted that the Firehose name must be start with “amazon-apigateway-“
3. Enable Custom Access Logging for APIGW
    1. Select your API in APIGW console
    2. Select one of your Stages, and Click “Logs/Tracing” Tab
    3. In Custom Access Logging, check Enable Access Logging
    4. Enter the Firehose ARN in Access Log Destination ARN
    5. In Log Format, select JSON and edit the log format as the following to include apikey, then click Save Changes

{ "requestId":"$context.requestId", "ip": "$context.identity.sourceIp", "apiKey":"$context.identity.apiKey","httpMethod":"$context.httpMethod","resourcePath":"$context.resourcePath", "status":"$context.status"}


4. Select “API Keys”  on the left panel
    1. Click Action -> Create API key
    2. Fill the Name of api key and generate a api key
5. Select “Usage Plans”
    1. Click Create
    2. File the Name of usage plan, setup your throttling if needed and click Next
    3. Click “Add API Stage” and select your API stage then click Next
    4. Click “Add API Key to Usage Plan” to associate the previously created api key with the api stage
6. Open a terminal in your laptop and input the following, the https url is your api url.

curl -X GET -H "x-api-key: your api key” -H "Content-Type:ation/json" -d '{"key":"val"}' https://g2oplii1kb.execute-api.cn-north-1.amazonaws.com.cn/test


7. Wait a few minutes (depends on your firehose batch setting), then go to the S3 bucket and you will find the api access log has been written to the bucket. And here is a example of the logs:

{ "requestId":"80aa583d-6caa-42e8-ab43-98e5c4341215", "ip": "54.221.61.34", "apiKey":"**********************************iFNdgS","caller":"-", "user":"-","requestTime":"15/Jul/2021:05:35:16 +0000", "httpMethod":"GET","resourcePath":"/", "status":"200","protocol":"HTTP/1.1", "responseLength":"1309" }{ "requestId":"841c5bb5-3336-4fe7-8361-412b396f9817", "ip": "54.221.61.40", "apiKey":"**********************************iFNdgS","caller":"-", "user":"-","requestTime":"15/Jul/2021:05:35:18 +0000", "httpMethod":"GET","resourcePath":"/", "status":"200","protocol":"HTTP/1.1", "responseLength":"1309" }{ "requestId":"bd089625-8c6a-4471-a7bc-cfa70e65eb14", "ip": "54.221.61.34", "apiKey":"**********************************iFNdgS","caller":"-", "user":"-","requestTime":"15/Jul/2021:05:35:19 +0000", "httpMethod":"GET","resourcePath":"/", "status":"200","protocol":"HTTP/1.1", "responseLength":"1309" }

8. Go to Athena, create a external table for the logs data and then query with sql to analysis the logs

CREATE EXTERNAL TABLE IF NOT EXISTS apigw_access_logs_db.test_logs (
  `requestID` string,
  `ip` string,
  `apiKey` string,
  `httpMethod` string,
  `resourcePath` string,
  `status` string 
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
WITH SERDEPROPERTIES (
  'serialization.format' = '1'
) LOCATION 's3://api-gateway-logs-bucket/2021/07/15/07/'
TBLPROPERTIES ('has_encrypted_data'='false');
