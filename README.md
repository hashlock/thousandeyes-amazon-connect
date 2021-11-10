# thousandeyes-amazon-connect

This reference project illustrates how to integrate ThousandEyes metrics directly into an Amazon Connect streams endpoint, like the softphone. It includes the following projects:

* **client** - example client side code that shows how to display ThousandEyes health metrics alongside Amazon Connect Streams soft phone. Includes sample HTML and CSS code that can be modified by the end user to fit their Amazon Connect client side deployment.
  
* **thousandeyes-metrics-service** - an AWS CloudFormation template that will deploy an API gateway service to allow the querying ThousandEyes API from the client frontend. This  to query ThousandEyes metrics. This project includes a CloudFormationTemplate and Labmda-API Gateway service.

# Setting Up ThousandEyes
The following steps assume you have a ThousandEyes account. If you do not, you can sign-up for a 14-day free trial at https://www.thousandeyes.com/signup/.

## Deploy and Configure ThousandEyes Endpoint Agent
First you'll want to setup your ThousandEyes monitoring by deploying ThosuandEyes Endpoint Agents. ThousandEyes Endpoint Agents monitor network and app performance across the internet as well as collect device health. With Endpoint Agents installed on your Amazon Connect users' devices, you'll be able to quickly identify any network or device issues that may be impacting a user's experience of Amazon Connect.

### Deploy ThousandEyes Endpoint Agents
Follow the steps in this [Endpoint deployment guide](https://docs.thousandeyes.com/product-documentation/global-vantage-points/endpoint-agents/quick-guide-on-endpoint-agent#configuring-and-deploying-the-endpoint-agent) to install the ThousandEyes Endpoint agent on any Windows or MacOS based user device.
1) For each deployed agent, note the Agent Name. You will use this when setting up the client widget below. The agent name is typically the device host name. For example: `hans-mac-us`

### Configure ThousandEyes to Monitor Amazon Connect Service
Once you've deployed at least one ThousandEyes Endpoint Agent you'll want to configure ThousandEyes to monitor Amazon Connect from those agents. To do this: 
1) Log in to your ThousandEyes account at https://app.thousandeyes.com
2) Select Endpoint Agent --> [Test Settings](https://app.thousandeyes.com/endpoint/test-settings/) 
3) Select "Add New Test" and create a new "Web - HTTP Server" test
4) In the **URL** field enter the URL of your Amazon Connect service. For example `https://acmebusiness.my.connect.aws`.  
5) Name the test `AmazonConnect` or something similar. You will use this name when setting up the client widget (see below)
![picture 6](readme/4afef72d2e22df4fa144f4786af74bc24416362565f1d3d2b411539eaeb50157.png)  
6) In the **Agents** field select the Endpoint Agents you installed in the previous step. 
![picture 2](readme/21828ef52af6622212d8453046bd3d395245bf391fe90b968538968978af57ab.png)  
7) Select **Add New Test** in the lower right.
8) Also note the Account Group name that you created this test in, you will use it below.
### Enable ThousandEyes Session Monitorintg
Next you'll want to enable session monitoring when the user is using the Amazon Connect web client. 
1) Click on **Browser Session Settings** --> **Monitored Domain Sets** --> Add New Monitored Domain Set
![picture 3](readme/886974b69ba1a66a50dd5e4bac96bf3a05201487993581d9f3f2a1abbd1fcea1.png)
2) Enter the web domain that your users browse to when using Amazon Connect. For example - if your users access Amazon Connect CCP via your Salesforce instance, you might add something like `*.salesforce.com`.  
![picture 4](readme/7b174a617a0bdf4cff1eace3b8a94bab5dc8b96f1445867cb5c2724fa838752c.png)  
3) Click **Add**
 
ThousandEyes will now start running network monitoring to your Amazon Connect instance from each of the Endpoint Agents you installed and specified. It will also collect device health metrics that can be used alongside the network monitoring to quickly troubleshoot any issues that impact Amazon Connect services when users are using the Amazon Connect CCP soft phone.
# Deploy the ThousandEyes Metrics Service
Next, you'll deploy the ThousandEyes metrics service to you AWS cloud environment. This service will allow querying live metrics data from ThousandEyes for your Amazon Connect users. That data can then be displayed using the ThousandEyes client widget on each end-user device alongside the Amazon Connect CCP.  
### Get your ThousandEyes API Token
First you'll need to get your ThousandEyes API token which will be used to query metrics from ThousandEyes.  
   1) Log in to ThousandEyes and click on Account Settings -> Users and Roles -> Profile.
   2) Your Basic Auth token is listed on the bottom of the page. If you do not have one yet, click "Regenerate"

### Deploy the CloudFormation template
Next, you'll need to deploy the service that will allow querying ThousandEyes metrics from your Amazon Connect streams client.
   1) Log in to your AWS account
   2) Click on the following CloudFormation template link to deploy the ThousandEyes metrics service in your AWS account. https://console.aws.amazon.com/cloudformation/home#/stacks/new?templateURL=file://thousandeyes-amazon-connect/aws-deploy.json
   3) Fill in the following:
      1) ThousandEyes Username (email)
      2) ThousandEyes API Token (see above)
   4) Click deploy
   5) Once deployed - copy the public API endpoint. Example: https://jur5bq3i8a.execute-api.us-east-1.amazonaws.com/dev/endpoint 

# Embed the ThousandEyes Client Widget
Lastly, you can use the included example client code to embed a ThousandEyes metrics widget on the same webpage that you embed the Amazon Connect CCP. 
1) The **/client** folder contains an example `index.html` file and javascript that shows how to embed the ThousandEyes widget alongside the Amazon Connect CCP. Open the `index.html` file.
1) Copy the url of your Amazon Connect instance and paste it into the `instanceURL` variable in `index.html`.
1) Copy the url from the above CloudFormation template deployment and paste it into the `metricsURL` variable in `index.html`.
For example:
```html
      <div id="ccpContainer1">
        <script>
          var instanceURL = "https://acmebusiness.my.connect.aws/connect/ccp#/";
          var metricsURL = "https://bqnoz0b8ol.execute-api.us-east-1.amazonaws.com/v1/";
          var container = document.getElementById("ccpContainer");
          var ccpURL = instanceURL;
          init();
        </script>
      </div>
```
1) Lastly, you'll need to specify the ThousandEyes account, test, and agent to pull metrics from. To do this, filling in the following variables in `index.html`:
```javascript
          var teAccount = "MyAccountGroup";
          var teTest = "AmazonConnect";
          var agent = "hans-mac-us";
```
2) The above code is an example. You can copy the contents of `index.html` as well as the `/js/common.js` file to your web server.

You should now be able to display the ThousandEyes widget on your Amazon Connect client webpage. Here's an example of the widget embedded below the default Amacon Connect CCP widget looks like:

![picture 5](readme/234f11324926ac530d2794d6f96d9f69dacd3fcc2cfa9205b2e873feaa6f09fc.png)  