# Develop and Secure APIs with Apigee X: Challenge Lab

## Task 1. Proxy the Cloud Translation API

Requirements:
1. In the Google Cloud Console, confirm that the Cloud Translation API is enabled in the API Library.
> Navigation menu > **API & Services** > **Enabled APIs & services**.
> Or, open cloud shell and run `gcloud services enable translate.googleapis.com`

2. Create a service account for the API proxy named apigee-proxy, and grant it the role Logging > Logs Writer.

> From the Navigation menu (Navigation menu icon), navigate to **IAM & Admin > Service Accounts.**
> Click **+ Create service account**.
> For Service account name, specify the following: `apigee-proxy`
> Click **Create and Continue**.
> Under **Permissions (optional)**, for **Select a role**, select **Logging > Logs Writer**.
> The Apigee API proxy will write log messages to Cloud Logging.
> Click **Done**.

### Create API Proxy
3. In the Google Cloud Console,from the Navigation menu, select Apigee to open the Apigee UI and create your API proxy.
4. The API proxy should be a reverse proxy named translate-v1, with a base path of /translate/v1.
5. The API proxy's target is the HTTP URL for the basic version of the Cloud Translation API (https://translation.googleapis.com/language/translate/v2).
6. Do not add authorization, CORS, or a quota using the Common Policies page of the proxy wizard.
7. On the summary page, create the API proxy by leaving the settings at their defaults.
8. Add an Authentication section to the default TargetEndpoint, causing an access token to be sent with every backend request. Use a GoogleAccessToken element with a Scope of https://www.googleapis.com/auth/cloud-translation.
Note: Edit the proxy and in the Develop tab, under Target endpoints section, edit the default.xml file.

> To open the Apigee console, in the Google Cloud console, in the Search field, enter Apigee and select **Apigee** the Product.
> In the navigation menu, select **Proxy development > API proxies**.
> To create a new proxy using the proxy wizard, click **Create**.
> For **Proxy template**, leave selection as **Reverse proxy (Most common)** under **General template**.
> Specify the following for the Proxy details:

| Property | Value |
|----------|-------|
| Proxy name | translate-v1
| Base path	| /translate/v1
| Target (existing API) | https://translation.googleapis.com/language/translate/v2

> Click **Next**, then **Create**.
> Once the API proxy is deployed, click on the **Develop** tab.
> In the left pane, under **Target endpoints**, click **default** to open the configuration file (default.xml).
> In the **default.xml** code editor, replace the <HTTPTargetConnection> element:
```xml
<HTTPTargetConnection>
    <URL>https://translation.googleapis.com/language/translate/v2</URL>
    <Authentication>
        <GoogleAccessToken>
            <Scopes>
                <Scope>https://www.googleapis.com/auth/cloud-translation</Scope>
            </Scopes>
        </GoogleAccessToken>
    </Authentication>
</HTTPTargetConnection>
```
> Click **Save**

9. Use the Cloud Shell script in lab instructions to confirm that the Apigee runtime is completely installed.
10. Save and deploy the translate-v1 proxy to the eval environment using the following service account:
`apigee-proxy@PROJECT.iam.gserviceaccount.com`
> Click **Deploy**.
> For **Environment**, select **eval**.
> For **Service account**, specify the service account provided in lab instruction: `apigee-proxy@PROJECT.iam.gserviceaccount.com`
> Click **Deploy**, then **Confirm**.
> Wait for the API proxy to be deployed. Under the **Overview** tab, the Status will change to a green check mark when deployment is complete.

## Task 2. Change the API request and response

### Property Set
1. Create a property set named **language.properties**.
> In the translate-v1 API proxy navigation menu, click on the **+** sign next to **Resources** to add resources
  - For **Resource type**, select **Property Set** 
  - For **Source**, select **Create new file**
  - For **Resource name**, enter **language**
  - Click **Add**
> Click on **language** under **Resources > properties** to open the property set editor. In the coding pane, add
```
output=es
caller=en
```
### POST /translate endpoint
2. In the proxy endpoint, create a path and verb conditional flow for the *POST /* resource. Name it *translate*.
> In the API proxy navigation menu, click on **default** under **Proxy endpoints**.
> On the right side panel, to the right of **Proxy Endpoint: default /translate/v1**, click on the **+** to add conditional flow.
> In the **Add conditional flow** dialog, enter the following:
  - For **Select Flow Type**, select **Condition flow**
  - For **Flow name**, enter `translate`
  - For **Condition type**, select `Path and verb`
  - For **Path**, enter `/`
  - For **Verb**, select `POST` 
> Click **Add**

### /languages endpoint
3. In the proxy endpoint, create a path (no verb) conditional flow for the */languages* resource. Name it *getLanguages*. 
> On the right side panel, to the right of **Proxy Endpoint: default /translate/v1**, click on the **+** to add conditional flow.
> In the **Add conditional flow** dialog, enter the following:
  - For **Select Flow Type**, select **Condition flow** 
  - For **Flow name**, enter `getLanguages`
  - For **Condition type**, select `Path`
  - For **Path**, enter `/languages`
> Click **Add**

### AM-BuildTranslateRequest
4. Create an AssignMessage policy named `AM-BuildTranslateRequest` to create the backend request used in the translate conditional flow.

> Check that the right side panel is showing **Proxy Endpoint: default /translate/v1**. Under **Request**, click on the **+** to the right of **translate** to add a policy step.
> In the **Add policy step** dialog, select **Create new policy**.
> In the **Select policy** field, filter for **Assign Message** under **Mediation**.
> For **Name** and **Display name**, enter `AM-BuildTranslateRequest`.
> Click **Add**.
> Click on the newly created `AM-BuildTranslateRequest` policy under **translate** to look at the policy's XML configuration.
> In the coding pane, replace the existing code with the following:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="AM-BuildTranslateRequest">
  <DisplayName>AM-BuildTranslateRequest</DisplayName>
  
  <!-- Read text from request body-->
  <AssignVariable>
    <Name>text</Name>
    <Template>{jsonPath($.text,request.content)}</Template>
  </AssignVariable>
  
  <!-- Determine target language-->
  <AssignVariable>
    <Name>language</Name>
    <Template>{firstnonnull(request.queryparam.lang,propertyset.language.output)}</Template> 
  </AssignVariable> 

  <Set>
    <!-- Build backend payload -->
    <Payload contentType="application/json">
      {
        "q": "{text}", 
        "target": "{language}"
      }
    </Payload>
  </Set>
  <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
  <AssignTo createNew="false" transport="http" type="request"/>
</AssignMessage>
```

References:
[AssignVariable element](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/assign-message-policy#assignvariable)
[jsonPath, message template](https://docs.cloud.google.com/apigee/docs/api-platform/reference/message-template-intro#json-path-function)
[firstnonnull function](https://docs.cloud.google.com/apigee/docs/api-platform/reference/message-template-intro#null-coalescing-function)
[access propertySet values](https://docs.cloud.google.com/apigee/docs/api-platform/cache/property-sets#access-values)
[Payload child element of <Set> that defines message body for request or response](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/assign-message-policy#set-payload)
[Assign to element](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/assign-message-policy#assignto)

### AM-BuildTranslateResponse
5.Create an AssignMessage policy named AM-BuildTranslateResponse under translate conditional flow to create the response for the caller using the Translation API response.
> Check that the right side panel is showing **Proxy Endpoint: default /translate/v1**. Under **Response**, click on the **+** to the right of **translate** to add a policy step.
> In the **Add policy step** dialog, select **Create new policy**.
> In the **Select policy** field, filter for **Assign Message** under **Mediation**.
> For **Name** and **Display name**, enter `AM-BuildTranslateResponse`.
> Click **Add**.
> Click on the newly created `AM-BuildTranslateResponse` policy under **translate** to look at the policy's XML configuration.
> In the coding pane, replace the existing code with the following:
```xml

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage continueOnError="false" enabled="true" name="AM-BuildTranslateResponse">
  <DisplayName>AM-BuildTranslateResponse</DisplayName>

  <AssignVariable>
    <Name>translated</Name>
    <Template>{jsonPath($.data.translations[0].translatedText,response.content)}</Template>
  </AssignVariable>
  
  <Set>
    <Payload contentType="application/json">{"translated":"{translated}"}</Payload>
  </Set>
  
  <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
  <AssignTo createNew="true" transport="http" type="response"/>
</AssignMessage>
```

### AM-BuildLanguagesRequest
6. Create an AssignMessage policy named AM-BuildLanguagesRequest to create the backend request used in the getLanguages conditional flow.
> Check that the right side panel is showing **Proxy Endpoint: default /translate/v1**. Under **Request**, click on the **+** to the right of **getLanguages** to add a policy step.
> In the **Add policy step** dialog, select **Create new policy**.
> In the **Select policy** field, filter for **Assign Message** under **Mediation**.
> For **Name** and **Display name**, enter `AM-BuildLanguagesRequest`.
> Click **Add**.
> Click on the newly created `AM-BuildLanguagesRequest` policy under **getLanguages** to look at the policy's XML configuration.
> In the coding pane, replace the existing code with the following:
```xml
<AssignMessage continueOnError="false" enabled="true" name="AM-BuildLanguagesRequest">
  <DisplayName>AM-BuildLanguagesRequest</DisplayName>
  <AssignVariable>
    <Name>targetLanguage</Name>
    <Template>{firstnonnull(request.queryparam.lang, propertyset.language.caller)}</Template>
  </AssignVariable>
  <Set>
    <Verb>POST</Verb>
    <Payload contentType="application/json">{"target":"{targetLanguage}"}</Payload>
  </Set>
  <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
  <AssignTo createNew="true" transport="http" type="request"/>
</AssignMessage>
```
[AssignVariable element](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/assign-message-policy#assignvariable)
[Authentication element](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/assign-message-policy#set-auth)

### JS-BuildLanguagesResponse
7. Create a JavaScript policy named JS-BuildLanguagesResponse under getLanguages conditional flow to create the response for the caller. 
> Check that the right side panel is showing **Proxy Endpoint: default /languages/v1**. Under **Response**, click on the **+** to the right of **getLanguages** to add a policy step.
> In the **Add policy step** dialog, select **Create new policy**.
> In the **Select policy** field, filter for **JavaScript** under **Extension**.
> For **Name** and **Display name**, enter `JS-BuildLanguagesResponse`.
> For **Javascript file**, select **Create new resource**.
> In the Add resource section, specify the following:
  - For **Source**, select **Create new resource**
  - For **Resource name**, enter `buildLanguagesResponse.js`
> Click **Add**.
> Select the newly created `buildLanguagesResponse.js` under Javascript file, then click **Add**.
> Click on the newly created `buildLanguagesResponse.js` under **Resources > jsc** to open the policy's javascript code.
> In the coding pane, add the following:
```javascript

// get the flow variable 'response.content'
var payload = context.getVariable("response.content");

// parse the payload into the response payload object
var payloadObj = JSON.parse(payload);

// select the languages array from the payloadObj
var newPayload = JSON.stringify(payloadObj.data.languages);

// convert the language array back into JSON
context.setVariable("response.content",newPayload);
```


Reference:
[JavaScript policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/javascript-policy)

Save and Deploy the API proxy to the eval environment using the same service account as in Task 1 step 10.
Test the API.
> Make sure you have an SSH connection to **apigeex-test-vm** in Cloud Shell. Refer to step 12 in Task 1.
> Test the API according to lab instructions.


## Task 3. Add API key verification and quota enforcement
Access to this API should be limited to approved applications, so you will add a VerifyAPI key policy, as well as a Quota policy to limit the number of requests.

### Requirements:
1. Create an API product with a name and display name of *translate-product*. This API product should have public access, automatically approve access requests, and be available in the eval environment.

### Create API product
> On the Apigee navigation menu, select **Distribution > API Products**.

> To create a new API product, click **+CREATE**.

> In the **Product details** pane, specify the following:

| Property |	Value |
|----------|-------|
| Name	| translate-product
| Display Name |	translate-product
| Environment |	select eval and click **OK**
| Access |	select Public

> Leave **Automatically approve access requests** selected.

2. Add an operation to the *translate-product* API product. The operation should allow access to the *translate-v1* proxy and use a path of */*, which allows access to any request, including /. Allowed methods are *GET* and *POST*. Add an operation quota setting of 10 requests per 1 minute.

> In the **Operations** section, click **+Add an Operation**.
> Specify the following:

| Name |	Value |
|-----|-------|
| API Proxy |	select the "translate-v1" API proxy
| Path |	/
| Methods |	select GET and POST
| Api quota limit | 10
| Time interval | 1
| Time unit |	minute
> Click **Save**
> On the main API product creation page, click **Save** at the top of the page.

### Create developer
3. Create a developer with the email *joe@example.com*. Choose your own first name, last name, and username.

> On the Apigee navigation menu, click **Distribution > Developers**.

To create a new app developer, click **+CREATE**.

Specify the following:

| Property |	Value |
|----------|-------|
| First Name |	Joe
| Last Name	| Developer
| Email |	joe@example.com
| Username |	joe

> Click **ADD** to create the app developer.

### Create app
4. Create an app called *translate-app*, and enable the *translate-product* API product for it. You will need to associate it with your *joe@example.com* developer.

> On the Apigee navigation menu, click **Distribution > Apps**.

>To create a new app, click **+CREATE**.

>In the **App details** pane, specify the following:

| Property |	Value |
|----------|-------|
| Name	| translate-app
| Developer |	select Joe Developer

>In the **Credentials** pane, click on **+ ADD PRODUCTS**, select **translate-product**, and then click **OK** and **Add** to add.

>Under **Product** click on checkbox beside translate-product and click on **APPROVE**

>Click **Create** to create the app.

> On the app details page, under **Credentials**, copy the value of the *key*. In Cloud Shell, save this value for use in the test case where KEY=<api key>: `export KEY=your_api_key_value`

CLI command to get API key:
```
export PROJECT_ID=$(gcloud config list --format 'value(core.project)' 2>/dev/null)
echo "PROJECT_ID=${PROJECT_ID}"
export API_KEY=$(curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" -X GET "https://apigee.googleapis.com/v1/organizations/${PROJECT_ID}/developers/joe@example.com/apps/translate-app" | jq ".credentials[0].consumerKey" --raw-output)
echo "KEY=${API_KEY}"
```

### Add VerifyAPIKey policy
5. Add a VerifyAPIKey policy named *VA-VerifyKey* to the proxy endpoint preflow. The API key should be required for every request, and should be sent in using the Key header.

> On the Apigee navigation menu, select **Proxy development > API Proxies**, and then click *translate-v1*.

> Click the **Develop** tab.

> In the Navigator menu for the proxy, in the **Proxy Endpoints** section, click **PreFlow**.

> In the Flow pane, click the **+ Add Policy Step** button to the right of **PreFlow** in the **Request** column.

> Select **Create new policy** and under **select policy** in the *Security* section, select **Verify API Key**, and then set the Display Name and Name to *VA-VerifyKey*.

>Click **Add**.

> Click on the name **VA-VerifyKey**.

> In the **VA-VerifyKey.xml** code pane, the **APIKey** element indicates where the API key is provided in the request.

> In the **APIKey** element, replace *request.queryparam.apikey* with *request.header.Key*.

### Add quota policy
6. Add a Quota policy named *Q-EnforceQuota* to the proxy endpoint preflow.

The policy should include the steps:

- Use a type of *calendar*. The calendar type requires a *StartTime* element.
- Specify *UseQuotaConfigInAPIProduct* to use the quota from the API product, with a default quota of 5 requests every one hour if the API product does not specify quota settings.
- Set *Distributed and Synchronous* to true, and remove the *AsynchronousConfiguration* element.

> In the Navigator menu for the proxy, in the **Proxy Endpoints** section, click **PreFlow**.

>In the Flow pane, click the **+ Add Policy Step** button to the right of **PreFlow** under **Request**.

> Select **Create new policy**. When asked to **select policy**, select **Quota** in the *Traffic Management* section, and then set the **Display Name** and **Name** to *Q-EnforceQuota*.

> Click **Add**.

> Click on the policy **Q-EnforceQuota**.

> The Quota configuration is shown in the Code pane.

> Change the Quota configuration to:
```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Quota continueOnError="false" enabled="true" name="Q-EnforceQuota" type="calendar">
  <DisplayName>Q-EnforceQuota</DisplayName>
  
    <UseQuotaConfigInAPIProduct stepName="VA-VerifyKey">
        <DefaultConfig>
            <Allow>5</Allow>
            <Interval>1</Interval>
            <TimeUnit>hour</TimeUnit>
        </DefaultConfig>
    </UseQuotaConfigInAPIProduct>
    <Distributed>true</Distributed>
    <Synchronous>true</Synchronous>
    <StartTime>2025-01-01 00:00:00</StartTime>
</Quota>
```
[Quota policy](https://docs.cloud.google.com/apigee/docs/api-platform/reference/policies/quota-policy)

> Click **Save**.If you are notified that the proxy was saved as a new revision, click SAVE AS NEW REVISION.

> Click **Deploy**

> Use **Service account** from Task 1 step 10.

>  Click **Deploy**, then **Confirm**.

>Click the **Overview** tab, and wait for the **eval** deployment status to show that the proxy has been deployed.

[CORS protocol](https://fetch.spec.whatwg.org/#cors-protocol)
[single origin security policy](https://www.w3.org/Security/wiki/Same_Origin_Policy)


## Task 4. Add message logging
Requirements:

1. Add a MessageLogging policy named *ML-LogTranslation* to the translate conditional flow. The policy must execute after the *AM-BuildTranslateResponse* step.

> Create a PostClientFlow. Policies attached to PostClientFlow execute after the response is returned to the client. This does not add latency to the client response time.
> In the Navigator menu, in the **Proxy Endpoints** section, click on **default**
> In the coding pane, add `<PostClientFlow/>` element immediately above the **HTTPProxyConnection** section, below the </Flows> element.


2. The policy should log to Cloud Logging.
The *LogName* value should be:
`projects/{organization.name}/logs/translate`
The logged message should have a *contentType* of *text/plain*, and the message contents should be:
`{language}|{text}|{translated}`
This message requires the *language, text*, and *translated* variables that were created in the *AM-BuildTranslateRequest* and *AM-BuildTranslateResponse* policies.

> In the Navigator menu for the proxy, in the **Proxy Endpoints** section, click **PostClientFlow**.
> In the Flow pane, click the **+ Add Policy Step** button to the right of **translate** in the **Response** column.
> > Select **Create new policy** and under **select policy** in the *Extension* section, select **Message Logging**, and then set the **Display Name** and **Name** to *ML-LogTranslation*.
> CLick **Add**.
> Shift the **ML-LogTranslation** policy step so that it executes after the *AM-BuildTranslateResponse* step.
> Click on the **ML-LogTranslation** policy.
> In the coding pane, replace the existing code with the following:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<MessageLogging continueOnError="false" enabled="true" name="ML-LogTranslation">
  <DisplayName>ML-LogTranslation</DisplayName>
  <CloudLogging>
    <LogName>projects/{organization.name}/logs/translate</LogName>
    <Message contentType="text/plain">{language}|{text}|{translated}</Message>
  </CloudLogging>
</MessageLogging>

## Task 5. Rewrite a backend error message
Requirements:
1. Add a FaultRules section to the default target endpoint. When the backend returns a 400 response, it will automatically evaluate any matching fault rules in the target endpoint.
Note: The UI Navigator menu on the left cannot be used to add a FaultRules section. You must add it in the target endpoint's XML configuration.
2. Add a *FaultRule* to the *FaultRules* section. The Condition of this FaultRule should be set so that it executes if *fault.name* is *ErrorResponseCode*.
A
> In the Navigation menu for the proxy, in the **Target Endpoint** section, click **default**.
> Edit the XML configuration in the coding pane. Add the following `<FaultRules>` element after <TargetEndpoint name="default">, before <PreFlow name="PreFlow">.
```xml
<TargetEndpoint name="default">

  <FaultRules>
    <FaultRule name="FR-Rewrite400Error">
      <Step>
        <Name>AM-BuildErrorResponse</Name>
      </Step>
      <Condition>fault.name=="ErrorResponseCode"</Condition>
    </FaultRule>
  </FaultRules>

<PreFlow name="PreFlow">
```

3. Create an *AssignMessage* policy named *AM-BuildErrorResponse* and attach it to the FaultRule.

> select **+** to the right of **Policies** to add a policy step.
> Filter for **AssignMessage** under **Mediation** section, and then set the **Display Name** and **Name** to *AM-BuildErrorResponse*.
> CLick **Create**, and then click on the **AM-BuildErrorResponse** policy.
> In the coding pane, replace the existing code with the following:
```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage name="AM-BuildErrorResponse">
  <Set>
      <Payload contentType="application/json">{ "error": "Invalid request. Verify the lang query parameter." }</Payload>
  </Set>
</AssignMessage>
```
> Click **Save** and deploy.
> Test API according to lab instructions.


The final *default.xml* for Proxy endpoint:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">
  <Description/>
  <FaultRules/>
  <PreFlow name="PreFlow">
    <Request>
      <Step>
        <Name>VA-VerifyKey</Name>
      </Step>
      <Step>
        <Name>Q-EnforceQuota</Name>
      </Step>
    </Request>
    <Response/>
  </PreFlow>
  <PostFlow name="PostFlow">
    <Request/>
    <Response/>
  </PostFlow>
  <Flows>
    <Flow name="translate">
      <Description/>
      <Request>
        <Step>
          <Name>AM-BuildTranslateRequest</Name>
        </Step>
      </Request>
      <Response>
        <Step>
          <Name>ML-LogTranslation</Name>
        </Step>
        <Step>
          <Name>AM-BuildTranslateResponse</Name>
        </Step>
      </Response>
      <Condition>(proxy.pathsuffix MatchesPath "/") and (request.verb = "POST")</Condition>
    </Flow>
    <Flow name="getLanguages">
      <Description/>
      <Request>
        <Step>
          <Name>AM-BuildLanguagesRequest</Name>
        </Step>
      </Request>
      <Response>
        <Step>
          <Name>JS-BuildLanguagesResponse</Name>
        </Step>
      </Response>
      <Condition>(proxy.pathsuffix MatchesPath "/languages")</Condition>
    </Flow>
  </Flows>
  <PostClientFlow/>
  <HTTPProxyConnection>
    <BasePath>/translate/v1</BasePath>
    <Properties/>
  </HTTPProxyConnection>
  <RouteRule name="default">
    <TargetEndpoint>default</TargetEndpoint>
  </RouteRule>
</ProxyEndpoint>
```

The final *default.xml* for Target endpoint:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TargetEndpoint name="default">
  <FaultRules>
    <FaultRule name="FR-Rewrite400Error">
      <Step>
        <Name>AM-BuildErrorResponse</Name>
      </Step>
      <Condition>fault.name=="ErrorResponseCode"</Condition>
    </FaultRule>
  </FaultRules>
  <PreFlow name="PreFlow">
    <Request/>
    <Response/>
  </PreFlow>
  <PostFlow name="PostFlow">
    <Request/>
    <Response/>
  </PostFlow>
  <Flows/>
  <HTTPTargetConnection>
    <URL>https://translation.googleapis.com/language/translate/v2</URL>
    <Authentication>
      <GoogleAccessToken>
        <Scopes>
          <Scope>https://www.googleapis.com/auth/cloud-translation</Scope>
        </Scopes>
      </GoogleAccessToken>
    </Authentication>
  </HTTPTargetConnection>
</TargetEndpoint>
```
