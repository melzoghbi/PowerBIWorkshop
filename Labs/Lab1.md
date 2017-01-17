# Lab 1: Build Real-Time Analytics with Power BI and IoT Hub

## Overview ##
In this module you will build the Azure back-end for an automotive parts supplier called "Parts Unlimited".
The challenge here is to process millions of events from concurrent users connected from different devices across the globe. 
With **Azure Event Hubs** or **Azure IoT Hub** you can process large amounts of event data from connected devices and applications. 
These are managed services that ingest events with elastic scale to accommodate variable load profiles and the spikes caused by intermittent connectivity.
In this module we are going to be using IoT hubs to ingest our device data.  
After you collect data into IoT Hubs, you can store the data using a storage cluster or transform it using a real-time analytics provider. 
**Azure Stream Analytics** is integrated out-of-the-box with Azure IoT Hubs to ingest millions of events per second. 
Stream Analytics processes ingested events in real-time, comparing multiple streams or comparing streams with historical values and models. 
It detects anomalies, transforms incoming data, triggers an alert when a specific error or condition appears in the stream, and displays this real-time data in your dashboard. 
For this scenario, you will use **Stream Analytics** to process and spool data to Blob Storage and Power BI.

### Objectives ###
In this module, you'll:
- Create an **IoT Hub**
- Register your device with the **IoT Hub**
- Use a device simulator to generate data
- Use **Stream Analytics** to process data in near-real time and spool data to **Blob Storage** and **Power BI**
- Create a sample **Power BI** dashboard


The following is required to complete this module:
- [Visual Studio Community 2015][1] or greater
- [Microsoft Azure Storage Explorer][2] or any other tool to manage Azure Storage such as Cloud Explorer in Visual Studio 2015.
- [PowerBI.com Account][3]
- An active Azure subscription

[1]: https://www.visualstudio.com/products/visual-studio-community-vs
[2]: http://storageexplorer.com/
[3]: https://github.com/paolosalvatori/ServiceBusExplorer
[4]: https://app.powerbi.com/signupredirect?pbi_source=web

## Exercises ##
This module includes the following exercises:

1. [Creating and integrating IoT Hubs](#Exercise1)
1. [Using Stream Analytics to process your data](#Exercise2)
1. [Visualizing your data with Power BI](#Exercise3)

Estimated time to complete this module: **75 minutes**

### Exercise 1: Creating and integrating IoT Hubs ###

Azure IoT Hubs is an event processing service that provides event and telemetry ingress to the cloud at massive scale, with low latency and high reliability. 
This service, used with other downstream services, is particularly useful in application instrumentation, user experience or work flow processing, and Internet of Things (IoT) scenarios.
In this exercise, you will use Azure IoT Hubs to track the user behavior in your retail website when viewing a product and also when adding it to the cart.

#### Task 1 - Creating the IoT Hub ####

There is a fantastic online resource already written that details exactly how to create your IoT hub and you can find it here [Getting Started](https://azure.microsoft.com/en-gb/documentation/articles/iot-hub-csharp-csharp-getstarted/).  
The article then goes on to cover how you can register your device with the IoT Hub using C#.  We have already done this part for you.  From your IoT Hub though you will need to grab a few details and these are;

- The hub connection string (Settings | Shared access policies | iothubowner)
- The hub host name

Now we have an IoT hub ready to receive events from our device/s


#### Task 2 - Configuring and starting event generator application ####

In this task, you'll set up and run a console application that will randomly create and send events - such as add, view, checkout and remove - to your Event Hub. Later in this module, you'll visualize these events in Power BI.

1. Copy the IoT Hub connection string from Azure portal as explained in (#Task1) 
2. Open in Visual Studio the **PowerBIWorkshop.sln** solution located at **Source / Lab1** folder.

You will notice there are three projects

- CreateDeviceIdentity - this will register your device with the IoTHub and display the key that needs to be used in sending messages .
- ReadDeviceToCloudMessages - this reads the messages from IoT hub that has been sent using SimlulatedDevice app.
- SimulatedDevice - this is the simulated device that sends messages to IoT hub.

3. Open CreateDeviceIdentity project and update the connectionString in App.config file.
4. Run the "CreateDeviceIdentity" and take a note of the device key that will be displayed.

![Running Create Device Identity App](/Images/RegisteringDeviceIoTHub.PNG)

5. Set SimulatedDevice app as a startup project.
6. Open App.config file, update iotHubUri and deviceKey configurations.

```

 <appSettings>
    <add key="iotHubName" value="HubName.azure-devices.net"/>
    <add key="deviceId" value="myFirstDevice"/>
    <add key="deviceKey" value="DEVICEKEY"/>
  </appSettings>


```

7. Run SimulatedDevice app, the app starts to send messages to the created IoT Hub you created earlier.

Now we have created our IoT hub and the applications that register, send and receive messages from IoT hub.

### Exercise 2: Using Stream Analytics to Analyse your Data ###

Now that we have a stream of events, you'll set up a Stream Analytics job to analyze these events in real-time.
Azure Stream Analytics (ASA) is a fully managed, cost effective real-time event processing engine that helps to unlock deep insights from data. 
Stream Analytics makes it easy to set up real-time analytic computations on data streaming from devices, sensors, web sites, social media, applications, infrastructure systems, and more.

#### Task 1 - Creating Stream Analytics job ####

In this task, you'll set up a Stream Analytics job to analyze the events in real-time.

1. In the [Azure portal](https://portal.azure.com/), click **New** > **Data + Analytics** > **Stream Analytics job**.
Specify the following values, and then click **Create**:

	- **Job Name**: Enter a job name.
	- **Resource Group**:
		- *"Use Existing"*
		- Please use the same resource group as the previous resources.
	- **Location**: Select the region where you want to run the job. Consider placing the job and the event hub in the same region to ensure better performance and to ensure that you won't be paying to transfer data between regions.
	
1. The new job will be shown with a status of Created. Notice that the Start button is disabled. You must configure the job **Input**, **Output**, and **Query** before you can start the job.

#### Task 2 - Specifying Data Stream job Input ####

In this task, you'll specify a job Input using the IoT Hub you previously created.

1. In your Stream Analytics job topology click **Inputs**, and then click **Add**. The blade to create a new Input appears on the right.

1. Type or select the following values for each setting:

	- **Input Alias**: Enter a friendly name for this job input such as **IoTHubInput**. Note that you'll be using this name in the query later.
	- **Source Type**: Select Data Stream.
	- **Source**: IoT Hub.
	- **Subscription**: Use IoT Hub from current subscription
	- **IoT Hub**: Name of the IoT Hub
	- **Endpoint**: Messaging (other option is Operations monitoring)
	- **Shared access policy name**: service
	- **Consumer group**: $Default
	- **Event Serialization Format**: JSON.
	- **Encoding**: UTF8.

1. Click **Create**.

#### Task 2 - Specifying job Output by adding Power BI output to Stream Analytics ####

In this task, you'll add an output to your Stream Analytics job.

1. From the [Azure portal](https://portal.azure.com/), go to Stream Analytics and click the one you created.

1. Click the **STOP** button below. We need to stop it in order to add a new output.

1. Click **OUTPUTS** from the top of the page, and then click **Add Output**.

1. In the **Add an Output** dialog box, select **Power BI** and then click the right button.

1. In the **Add a Microsoft Power BI output**, supply a work or school account for the Stream Analytics job output. If you already have Power BI account, select **Authorize Now**. If not, choose **Sign up now**.

1. Next, provide the values for:

	- **Output Alias** – You can put any output alias that is easy for you to refer to. This output alias is particularly helpful if you decide to have multiple outputs for your job. In that case, you have to refer to this output in your query. For example, let’s use the output alias value “**TopSellingPowerBI**”.
	- **Dataset Name** - Provide a dataset name that you want your Power BI output to have. For example, let’s use “**datamodulepbi**”.
	- **Table Name** - Provide a table name under the dataset of your Power BI output. Let’s say we call it “**datamodulepbitable**”. Currently, Power BI output from Stream Analytics jobs may only have one table in a dataset.
	- **Workspace** - You can use the default, My Workspace.

1. Click **OK, Test Connection** and now your Power BI connection is completed.

1. Lastly, you should update your **Query** to use this output and **start** the job. 
 
1. _Explanation_: This query will help us get the product sales for each 5 minute intervals, which we can then aggregate in PowerBI.

	```sql

	WITH AllEvents AS (
SELECT
    productId, type, Count(productId) AS [totalSold]
FROM
    IoTHubInput TIMESTAMP BY EventDate
WHERE type = 'checkout'
GROUP BY
    productId, type, TumblingWindow(minute, 1)
)

SELECT a.productId, a.type, a.totalSold 
INTO TopSellingPowerBI 
FROM AllEvents a

	```


Below figure Stream Analytics job we created to accept IoT hub events and stream the output to Power BI dataset.

![Microsoft Azure stream analytics Job](/Images/StreamAnalyticsJob.PNG)


#### Task 3 - Creating the dashboard in Power BI ####

1. Go to [PowerBI.com](https://powerbi.microsoft.com/) and login with your work or school account. 
If the Stream Analytics job query outputs results, you'll see your dataset is already created:

1. For creating the dashboard, go to the **Dashboards** option and create a new Dashboard, e.g. **DataLab1Dashboard**.

1. Scroll down to the **Datasets** section. You will notice the two datasets (“datamodulepbi” in our current example) that we created in Stream Analytics.


1. Now click on the **datamodulepbi** dataset created by your Stream Analytics job. You will be taken to a page to create a chart on top of this dataset.

1. You should see our table (datamodulepbitable) show up, along with all the fields associated with it. Select the **Stacked Column Chart** icon from the **Visualizations** menu on the right. 

1. Drag the **title** column in the **Axis** field and the **totalsold** column into the **Value** field
	

1. You should see your stacked column chart appear. Next, Click on the **...** in the top right corner of the visual and select **Sort by totalsold**.
	

1. You should now see your report sorted by Top Selling Products
	

1. Now click on the **Format** section under the **Visualization** tab and expand the **Title** menu. Change the Title Text to **Current Top-Selling Products**.
	

1. Click **Save** found in the top right of the canvas. You can name it "Top Selling report".

1. On the chart, click on the **pin-looking icon** found in the top-right corner of the visual. 
It will ask you whether you want to create a new dashboard or pin it to an existing one. 
Select **Existing Dashboard** and select the dashboard we created in the earlier step (eg: **PowerBIWorkshopDashboard**).

1. Go to your dashboard, click on the **ellipsis (...)**  button at the top-right corner of the tile and click the **pen** button to edit the _Tile details_, select the **Display last refresh time** functionality and apply the changes.

![Stream Analytics Dashboard in Power BI](/Images/FinalReport_IoTHub.PNG)
	
---

## Summary ##

By completing this module, you should have:

- Created an **IoT Hub**
- Used **Stream Analytics** to process data in near-real time and spool data to **Blob Storage** and **Power BI**
- Created sample **Power BI** charts & graphs