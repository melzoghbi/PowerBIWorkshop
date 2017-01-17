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

#### Task 2 - Create a Storage account ####

One of the data repositories for our **Azure Streaming Analytics** jobs is blob storage.  We are going to need to setup and account now.
You can see exactly how to setup a storage account [here](https://azure.microsoft.com/en-gb/documentation/articles/storage-create-storage-account/).  
We are going to create a **General Purpose Storage Account**  The article we just mentioned has a good discussion on the differences between the different types of account as well as the difference between hot and cold storage.
We won't need the details from this storage account until we start to use **Azure Streaming Analytics**

Suggested Storage Account Name: labstr`<uniqueSuffix>`

>**Note:** Please try and use the same resource group created during the first resource creation for this resource. 

Now we have things setup we can go ahead and register our device as well as start sending data to our IoT Hub.

#### Task 3 - Configuring and starting event generator application ####

In this task, you'll set up and run a console application that will randomly create and send events - such as add, view, checkout and remove - to your Event Hub. Later in this module, you'll visualize these events in Power BI.

1. Copy the IoT Hub connection string from Azure portal as explained in (#Task1) 
2. Open in Visual Studio the **PowerBIWorkshop.sln** solution located at **Source / Lab1** folder.

You will notice there are three projects

- CreateDeviceIdentity - this will register your device with the IoTHub and display the key that needs to be used in sending messages .
- ReadDeviceToCloudMessages - this reads the messages from IoT hub that has been sent using SimlulatedDevice app.
- SimulatedDevice - this is the simulated device that sends messages to IoT hub.

3. Open CreateDeviceIdentity project and update the connectionString in App.config file.
4. Run the "CreateDeviceIdentity" andtakea note of the device key that will be displayed.

[Running Create Device Identity App](/Images/RegisteringDeviceIoTHub.PNG)

5. Set SimulatedDevice app as a startup project.
6. Open App.config file, update iotHubUri and deviceKey configurations.


