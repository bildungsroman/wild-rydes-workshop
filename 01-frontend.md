# Frontend Content
In this step of the workshop you will create and deploy the *Wild Rydes* frontend. The frontend is composed of an AWS S3 bucket and an AWS Lambda function that is triggered on deployment which will populate the S3 bucket with the *Wild Rydes* static content.

## AWS Services

* [AWS S3](https://docs.stackery.io/docs/api/nodes/ObjectStore/)
* [AWS Lambda](https://docs.stackery.io/docs/api/nodes/Function/)
* [AWS Cloudformation](https://docs.aws.amazon.com/cloudformation/index.html)

## Instructions

> (Optional) If you use the VS Code IDE, parts of steps 1-4 can be completed in VS Code using the Stackery extension. Read the [VS Code setup instructions](vscode-setup-instructions.md) if you would prefer to work in the IDE rather than the browser. You will still need a Stackery account, as you need to log into Stackery when using the extension.

### 1. Add an Object Store resource
Add an *Object Store* resource (an AWS S3 Bucket) to serve the website content. With `stackery edit` running in the terminal, click the **Add Resource** button in the top right of the screen to reveal the resources menu. Then click on the *Object Store* resource to add it to the canvas and your application stack. Alternatively you can also drag and place the resource on the canvas.

![Add Object Store](./images/01-object-store.png)

Next, double-click on the Object Store resource on the canvas to edit its settings. Set the **CLOUDFORMATION LOGICAL ID** field to `FrontendContent`. Then click **ENABLE WEBSITE HOSTING** and leave the value of **INDEX DOCUMENT** as `index.html`. Be sure to click the button to save the Settings.

![Configure Object Store](./images/01-object-store-config.png)

### 2. Add a Function resource
Add a Function resource (an AWS Lambda Function) to update the website's static content. This function will copy the contents of a directory in the project source code to the Object Store we've just configured. You will also configure this Function resource to be triggered on every deployment of the stack.

From the *Add Resources* menu (found buy clicking *Add Resource*), click a Function resource to add it to the stack.

![Function Resource](./images/01-function.png)

Next drag a wire from the Function to the *FrontendContent* Object Store. **Make sure to drag the wire from the right end of the Function resource and connect it to the left end of the Object Store resource**. The line you are dragging should be a dotted line and not a solid line if you've done this correctly. This distinction is necessary because we want to indicate that the Function resource needs permissions to access the Object Store resource, not that the Object Store resource is an event trigger for the Function.

![Function Relationship](./images/01-function-relationship.png)

To tell if you've drawn the relationship correctly, double-click on the Function resource and scroll down to **ENVIRONMENT VARIABLES**. You should see the variables `BUCKET_NAME` and `BUCKET_ARN` defined.

![Function S3 Environmental Variables](./images/01-function-s3-env-vars.png)

Next in the Function's settings, for the **LOGICAL ID** field enter the value `PopulateFrontendContent`. Then update the **SOURCE PATH** field to `src/PopulateFrontendContent`. Stackery will create a scaffold for the function code on your local machine.

![Function Config](./images/01-function-config.png)

Scroll further down in the settings and check the **TRIGGER ON FIRST DEPLOY** and **TRIGGER ON EVERY DEPLOY** box. This will create an [AWS CloudFormation CustomResource](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cfn-customresource.html) in the stack that will trigger the function on deployments and updates. After you've done this, click the **Save** Button.

![Function Config Deploy](./images/01-function-config-deploy.png)

Your stack should now look like this:

![Commit Function changes](./images/01-initial-stack.png)


### 3. Edit function code locally

You will now locally edit the code of the _PopulateFrontendContent_ function.

If you browse the contents of your project directory you will notice the repository has a scaffold for the _PopulateFrontendContent_ Function resource in _src/PopulateFrontendContent_

```
$ tree stackery-wild-rydes
stackery-wild-rydes/
├── deployHooks
├── src
│   └── PopulateFrontendContent
│       ├── .stackery-config.yaml
│       ├── index.js
│       ├── package.json
│       └── README.md
├── .stackery-config.yaml
└── template.yaml
```

Before we do anything else, we need to get some source code from this workshop to go into that directory.

First, `git clone` this workshop to your computer. You will be copying code from the workshop repository into your own application stack. Open a new terminal tab or window in the same directory, then enter the following:

```
cd ..
git clone https://github.com/stackery/wild-rydes-workshop.git wild-rydes-workshop
```

Copy the following files and directories from the workshop to your application stack's directory.

* [src/PopulateFrontendContent/index.js](./src/PopulateFrontendContent/index.js)
* [src/PopulateFrontendContent/package.json](./src/PopulateFrontendContent/package.json)
* [src/PopulateFrontendContent/static/](./src/PopulateFrontendContent/static/)

You can do this by running the following commands on Linux or MacOS.

```bash
cp wild-rydes-workshop/src/PopulateFrontendContent/index.js stackery-wild-rydes/src/PopulateFrontendContent
```
```bash
cp wild-rydes-workshop/src/PopulateFrontendContent/package.json stackery-wild-rydes/src/PopulateFrontendContent
```
```bash
cp -R wild-rydes-workshop/src/PopulateFrontendContent/static stackery-wild-rydes/src/PopulateFrontendContent
```

You'll notice a new `static` directory in your `src/PopulateFrontendContent/` folder - that's going to be your app frontend that we will deploy in the next step.


### 4. Deploy the stack
You'll now deploy the *stackery-wild-rydes* stack to AWS. Stackery will package your code repository and deploy it using AWS CloudFormation.

We'll be using the `stackery deploy` command with the `interactive-setup` flag. This allows us to add our new stack to Stackery, create a new environment to deploy into, and even link our AWS account if needed. In the terminal, enter:

```bash
cd stackery-wild-rydes
stackery deploy --interactive-setup
```

Follow the prompts to continue: hit enter to keep the stack name *stackery-wild-rydes*, then select your AWS profile (if you only have one, it'll likely be called `default` and may be selected automatically). When asked if you would like to create a new environment, hit `y` and enter an environment name - we suggest `development`. It's normal to have multiple environments when building serverless apps, such as development, staging, and production.

Once your stack and environment have been created, the deployment process begins. This typically takes a few minutes. When finished, your readout should look something like this:

![01-cloudformation](./images/01-cli.png)


### 5. View the website

Now you can visit your Wild Rydes website that you have deployed.

Once the deployment is complete, open the Stackery App in your browser, and navigate to the __Stacks__ view tab:


![Deploy View](./images/01-stack-view.png)

Click on the *stackery-wild-rydes* stack. You will be taken to the __Overview__ tab. From there, click on the __View__ link, and you'll see your stack's deployed view:

![Deploy View](./images/01-deploy-view.png)

Double-click the *FrontendContent* Object Store resource to view its details. On the details slide-in click on the **Website Hosting Address** link to open the website.

![Deploy View](./images/01-deploy-link.png)



The website should appear, though it's missing resources it needs to be fully functional.

![Wild Rydes](./images/01-wild-rydes.png)

## Next Steps

Proceed to the next module in this workshop:

* [User Management](./02-user-management.md)

