# How to build a Dialogflow CX agent

This tutorial covers an end to end solution on how to create a sample Dialogflow CX agent with dynamic endpoints and connect to a website.  As an example, we will create a Virtual DMV agent that can assist the DMV customers to find the nearest DMV office based on their location.

Here are the main steps covered in this tutorial:

* Create a conversation script
* Create a simple agent with 1 flow 
- Create Internets, Parameters and Entities
- Create Webhooks and connect to the agent
- Embed the agent to a website

## Agent conversation Script

Before we create an agent we would need to create a sample scenario with which an end-user will interact with the agent. In this case, the end user is trying to find the closest DMV location. After we create the agent we can test it using the script. And example script for this conversation can be the following: 

**Caller: **

I’m looking for the nearest DMV office

**Agent:**

Sure, I can help you with that.  What is your zip code?

**Caller:**

My zip code is 64114.

**Agent:**

The closest DMV office is Santa Clara DMV. It is located at 3665 Flora Vista Ave, Santa Clara, CA 95051, United States. Is there anything else I can help you with?

**Caller:**

No, thank you!

## Create a static Agent

1. Before creating a Dialogflow agent, create a GCP Project. You can also use an existing project. Alternatively you can create a GCP project in Dialogflow CX interface. 
2. Go to [Dialogflow CX](https://dialogflow.cloud.google.com/cx) main page. Select a project and click on Create Agent. You can also import one of the pre-built agents and modify them in your project.
3. Create an agent called Virtual DMV and select the location and time zone. 
4. Create a new flow and name it Find DMV. To create a flow click on the + icon next to Flows on the upper left corner of the Build Panel. 
5. Now let’s create some intents.  To create an Intent click on Manage, then Intents, then Create. By default we have Default Welcome Intent and Default Negative Intent. The Default Welcome Intent is matched when the end-user starts a conversation. You can modify it as needed.  
6. Create a new Intent called closest.dmv.office.  In the Training Phrases section add the following phrases. Click Save

		I'd like to find a dmv office close to my house

		where is the dmv office

		nearest dmv office

		Closest dmv office

		where is the nearest dmv office?

		where is the closets dmv office?

		I'm looking for the nearest dmv office

		I'm looking for the dmv office

7. Similarly create 2 more intents. Create an intent called zip.code and add the following phrases. Here we can see the highlighted zip codes. Those are parameters. 	

	my zip is 87645

	my zip 93423

	87634

	Here is my zip code 98623

	zip 45321

	My zip code is 98432


8. Create parameters for this Intent. For this scenario we create a new parameter called zip_code.  To convert the entered zip codes into parameters, click on or highlight the zip code and select the entity @sys.zip-code from the list. Repeat this process for each training phrase. 

9. Now create Intent called end.call and add the phrases

	I'm good thanks

	all good

	that's all

	that'll be all

	no thanks

	no thank you

	no

	nothing


10. Now that we have our Intents, Entities and Parameters, we can start building the agent. Click on the Build menu at the top. At this point we have our Start page which was created by default. Now we can create additional pages.

11. Click on the + icon in the Pages panel and Add a page. Create 3 pages naming them DMV Location, Zip Code and End Call

12. Now we can create Routes. Click on the Start Page and Add a Route
	

13. First, you need to select an Intent or a Condition for a given route. In this case, select closest.dmv.office as the intent.

14. Go to the end of the page and select the next page which will follow the Start page, In this case it’s the page DMV Location. Click Save.

15. Now the flow is updated and DMV Location page comes after Start

16. Click on DMV Location Page and Edit the Fulfillment message as “Sure, I can help you with that.  What is your zip code?” Click Save.

17. Similar to the previous steps, click on the DMV Location page and Add a new Route. Select zip.code as the intent and Zip Code page as a transition page. Leave everything else as it is. Click Save.  

18. Click on the Zip Code page and Edit Fulfillment. In the Agent says section add the following message “ The closest DMV office is Santa Clara DMV. It is located at 3665 Flora Vista Ave, Santa Clara, CA 95051, United States. Is there anything else I can help you with?”
 -	 In the next section we will remove this message and connect a webhook to this page. The fulfillment message will be called directly from the webhook. 
19. In the same way click on the Zip Code and add a route. Select end.call as the intent and End Call as the page. Click Save. 
20. Click on the End Call page and add the following fulfillment message in the Agent says section. “Ok Have a nice day!”. 

21. At this point the flow is ready. It should look like this. 

22. You can test your static agent by clicking on the Test Agent on the upper right corner. Use the script for testing. 
23. Now the static agent is ready. In the next section we will add a webhook to call Google Places API to find the nearest DMV location based on the provided zip code. 

## Create a webhook and connect to the agent

In this section we will continue developing the agent and add some dynamic functionalities. 
We  will create a Cloud Function in the same project where our agent is built. This function will be called as a webhook in the Dialog Flow agent. Here are the steps covered in this section.

- Download code from github 
- Create an API Key
- Create a Cloud Function. This function does the following:
- Converts address to Zip Codes using  GeoCoding api
- Uses Google Places API to find nearest location based on the zip code
- Adds the json request response message in the function
- Create a webhook in Dialogflow 
- Add the webhook to the Zip Code page.
- Test the agent

1. Go to the Cloud Console and select the same project where the DialogFlow agent is deployed. 
2. Go to APIs & Services, Credentials, Create Credentials -> API Key and Create an API key. 
3. Download the code from this [Github repo](https://github.com/GPS-Demos/Workshop-DialogflowCX)
4. Go to the Cloud Functions and click Create Function
5. Name the function dmv_location and select the same region where your DialogFlow agent is. Leave the other default settings. Click Save then Next
6. Under Runtime select Node.js 10 and add getDMVLocation under Entry Point. This is the exported function name.  You need to make sure that the exported function name in your code and the Entry Point match.
7. Copy the code from Github and update index.js and  package.json files accordingly.
8. Update index.js file with the newly created API key
9. Replace API_KEY  in index.json file with your newly created API key value
10.Click Deploy. This will take a while. 
11. After the function is successfully deployed click on the function and go to the Trigger section. Copy the Trigger URL. We will use this in the agent. 
12. Now go back to the Dialogflow agent. Click on Webhooks, and Create New.
13. Add dmv_location as the Display name . Paste the Trigger URL copied in the previous step into the Webhook URL field. Leave other fields as default.  Click Save.
14. Go back to the Build section and click on the Zip Code page.  Edit Fulfillment. 
15 Remove the message under the Agent says field. Scroll down and check the Use Webhook checkbox. Then select the webhook dmv_location and create a Tag. Click Save. 
16. Now test the agent again. This time when the agent asks for a zip code, provide any valid zip code. And the agent will find the nearest DMV office and return the address.

## Embed the agent in a website

Now that we have a working agent, you can embed the agent into your existing website. To do so in the Dialogflow CS interface navigate to the Integrations and click Connect in the Dialogflow Messenger box. Copy the provided code and paste it in your website where you want the chatbot agent to appear. 





