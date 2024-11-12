# Salesforce Data Cloud auth process in MuleSoft

This is a simple Mule 4 application where you can see the process to authenticate to Data Cloud using only HTTP Requests.

For more examples and instructions to integrate MuleSoft and Data Cloud using the connector, please see [this repo](https://github.com/alexandramartinez/datacloud-mulesoft-integration).

Video explanation: [How to manually authenticate to Salesforce Data Cloud through a MuleSoft integration](https://youtu.be/iGlPOy_EMlE)

> [!IMPORTANT]
> Only use this manual authorization if the operation you are trying to access is not currently available in [Salesforce Data Cloud Connector - Mule 4](https://www.mulesoft.com/exchange/com.mulesoft.connectors/mule4-sdc-connector/). Otherwise, please use the connector instead.

## Similar repos

[![](https://github-readme-stats.vercel.app/api/pin/?username=alexandramartinez&repo=datacloud-mulesoft-integration&theme=default_repocard)](https://github.com/alexandramartinez/datacloud-mulesoft-integration)
[![](https://github-readme-stats.vercel.app/api/pin/?username=alexandramartinez&repo=mule-dynamodb-to-datacloud&theme=default_repocard)](https://github.com/alexandramartinez/mule-dynamodb-to-datacloud)

## Auth process explained

These are all the steps the Mule application is taking care of. If you intend to use this code, you only need to update the properties in [default.yaml](src/main/resources/default.yaml).

If you need a different call that's not `/api/v1/ingest/jobs` for the Data Cloud API (and is not currently available from the Data Cloud connector), then you just need to update this path in the last HTTP Request.

### Step 1: Retrieve the first token

Send a POST request to `https://login.salesforce.com/services/oauth2/token` with an `application/x-www-form-urlencoded` body containing the following information:

```json
{
  "client_id": "your-connected-app-clientId/key",
  "client_secret": "your-connected-app-clientSecret",
  "username": "salesforce-username",
  "password": "salesforce-password",
  "grant_type": "password"
}
```

> [!NOTE]
> In this example, I am using password authentication, but you can use others if needed. Learn more about this [here](https://help.salesforce.com/s/articleView?id=sf.remoteaccess_oauth_username_password_flow.htm&type=5).

You will receive a JSON response similar to this one:

```json
{
  "id": "https://login.salesforce.com/id/00Dx0000000BV7z/005x00000012Q9P",
  "issued_at": "1278448832702",
  "instance_url": "https://yourInstance.salesforce.com/",
  "signature": "0CmxinZir53Yex7nE0TD+zMpvIWYGb/bdJh6XfOH6EQ=",
  "access_token": "00Dx0000000BV7z!AR8AQAxo9UfVkh8AlV0Gomt9Czx9LjHnSSpwBMmbRcgKFmxOtvxjTrKW19ye6PE3Ds1eQz3z8jr3W7_VbWmEu4Q8TVGSTHxs",
  "token_type": "Bearer"
}
```

You will need to retrieve the `instance_url` and the `access_token` for the next call.

### Step 2: Retrieve Data Cloud's details

Take the `instance_url` you got from the previous call and concatenate the path `/services/a360/token` at the end. You should end up with something like:

```
https://yourInstance.salesforce.com/services/a360/token
```

Now make a POST request to that URL with an `application/x-www-form-urlencoded` body containing the following information:

```json
{
  "grant_type": "urn:salesforce:grant-type:external:cdp",
  "subject_token": "the access_token you got from the previous call",
  "subject_token_type": "urn:ietf:params:oauth:token-type:access_token"
}
```

You will receive a JSON response similar to the previous one. Once again, you will need to retrieve the `instance_url` and the `access_token` for the next call.

### Step 3: Call the Data Cloud API

Take the `instance_url` from the previous call and concatenate the path you want to call from Data Cloud's API. In my case, it was `/api/v1/ingest/jobs`. You also have to add `https://` at the beginning of the URL. You will use this new URL to call the Data Cloud API.

Apart from that, you have to make sure to send the Bearer token in the Authorization header. For that, make sure you send the `Authorization` header in the request and the value should be:

```
Bearer your-access_token
```

Make sure you leave a space between the word Bearer and the actual access_token from step 2.
