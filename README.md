# apigw-upload-in-s3 https://medium.com/swlh/processing-multipart-form-data-using-api-gateway-and-a-java-lambda-proxy-for-storage-in-s3-e6598033ff3e

Step 1: Use s3 bucket (s3:// ccds-call-volume-test/) and edit the CORS configuration of the bucket. This will allow GET and PUT request to interact with your bucket, from the web browser.

<?xml version=”1.0" encoding=”UTF-8"?>
<CORSConfiguration xmlns=”http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
 <AllowedOrigin>WRITE ORIGIN OR PUT * FOR TESTING</AllowedOrigin>
 <AllowedMethod>GET</AllowedMethod>
 <AllowedMethod>PUT</AllowedMethod>
 <AllowedHeader>WRITE ORIGIN OR PUT * FOR TESTING</AllowedHeader>
</CORSRule>
</CORSConfiguration>

Step 2: Create policies and role for lambda via AWS IAM, so that lambda can generate the URL. Then add the policy to a role for lambda, also add AWSLambdaBasicExecutionRole to the lambda role so that you can monitor your function on cloud watch.

 

Step 3 : Create a lambda function, attach the above role to it. The following code take the filename (key) from the API Gateway and generates a signed url.

const AWS = require('aws-sdk');
const s3 = new AWS.S3({signatureVersion: 'v4'});
exports.handler = async (event,context) => {
  const bucket = process.env.BUCKET_NAME;
  const key = event.key;
  const params = {
    Bucket: bucket,
    Key: key,
    ContentType: 'multipart/form-data’,
    Expires: 60
  };
  try{
    const signedURL = await s3.getSignedUrl(’putObject’, params);
    const response = {
        err:{},
        body:"url send",
        url:signedURL
    };
    return response;
  }catch(e){
        const response = {
        err:e.message,
        body:"error occured"
    };
      return response;
  }
};


Step 4: Create an API Gateway, and make this API a trigger to above created lambda function. Ensure the CORS settings of the API is updated. 

Step 5 : Test the file upload First make a post Request to the API, with filename (example.txt) as body of the request i.e. key: example.txt, the API will see this filename and lambda function will generate a URL according to the filename and send this URL back as response. We have to the send a put request to received URL with correct headers. Body of the request will be having the file itself as multipart/formdata.

const options = {
 headers: {
  "Content-Type": "multipart/form-data"
 }
};
// inside an async function
const res = await axios.put("URL",Your-File,options);
console.log(res);
