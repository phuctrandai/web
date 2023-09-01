# TechConf Registration Website

## Project Overview
The TechConf website allows attendees to register for an upcoming conference. Administrators can also view the list of attendees and notify all attendees via a personalized email message.

The application is currently working but the following pain points have triggered the need for migration to Azure:
 - The web application is not scalable to handle user load at peak
 - When the admin sends out notifications, it's currently taking a long time because it's looping through all attendees, resulting in some HTTP timeout exceptions
 - The current architecture is not cost-effective 

In this project, you are tasked to do the following:
- Migrate and deploy the pre-existing web app to an Azure App Service
- Migrate a PostgreSQL database backup to an Azure Postgres database instance
- Refactor the notification logic to an Azure Function via a service bus queue message

## Dependencies

You will need to install the following locally:
- [Postgres](https://www.postgresql.org/download/)
- [Visual Studio Code](https://code.visualstudio.com/download)
- [Azure Function tools V3](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#install-the-azure-functions-core-tools)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)

## Project Instructions

### Part 1: Create Azure Resources and Deploy Web App
1. Create a Resource group
2. Create an Azure Postgres Database single server
   - Add a new database `techconfdb`
   - Allow all IPs to connect to database server
   - Restore the database with the backup located in the data folder
3. Create a Service Bus resource with a `notificationqueue` that will be used to communicate between the web and the function
   - Open the web folder and update the following in the `config.py` file
      - `POSTGRES_URL`
      - `POSTGRES_USER`
      - `POSTGRES_PW`
      - `POSTGRES_DB`
      - `SERVICE_BUS_CONNECTION_STRING`
4. Create App Service plan
5. Create a storage account
6. Deploy the web app

### Part 2: Create and Publish Azure Function
1. Create an Azure Function in the `function` folder that is triggered by the service bus queue created in Part 1.

      **Note**: Skeleton code has been provided in the **README** file located in the `function` folder. You will need to copy/paste this code into the `__init.py__` file in the `function` folder.
      - The Azure Function should do the following:
         - Process the message which is the `notification_id`
         - Query the database using `psycopg2` library for the given notification to retrieve the subject and message
         - Query the database to retrieve a list of attendees (**email** and **first name**)
         - Loop through each attendee and send a personalized subject message
         - After the notification, update the notification status with the total number of attendees notified
2. Publish the Azure Function

### Part 3: Refactor `routes.py`
1. Refactor the post logic in `web/app/routes.py -> notification()` using servicebus `queue_client`:
   - The notification method on POST should save the notification object and queue the notification id for the function to pick it up
2. Re-deploy the web app to publish changes

## Monthly Cost Analysis
Complete a month cost analysis of each Azure resource to give an estimate total cost using the table below:

| Azure Resource | Service Tier | Monthly Cost |
| ------------ | ------------ | ------------ |
| *Azure Web App* | Basic Tier; 1 B3 (4 Core(s), 7 GB RAM, 10 GB Storage) x 730 Hours; Linux OS | $51.83 |
| *Azure Postgres Database* | Flexible Server Deployment, General Purpose Tier, 1 D2s v3 (2 vCores) x 730 Hours (Pay as you go), 100 GiB Storage, 0 Provisioned IOPS, 0 GiB Additional Backup storage - LRS redundancy, without High Availability | $156.15 |
| *Azure Service Bus*   | Basic tier: 1 million messaging operations | $0.05 |
| *Azure Storage Account*   | Queue Storage, General Purpose V1, LRS Redundancy, 100 GB Capacity, 1,000 Queue Class 1 operations, 1,000 Queue Class 2 operations | $5.22 |
| *Azure Function*  | Consumption tier, Pay as you go, 1024 MB memory, 100 milliseconds execution time, 1,000 executions/mo | $0.00 |
| *Total* |                                     | $213.25 |

## Architecture Explanation
This is a placeholder section where you can provide an explanation and reasoning for your architecture selection for both the Azure Web App and Azure Function.

- The Web App with the Basic tier is compatible for this application:
   + It doesn't require a high-performance machine, so the Basic tier is sufficient.
   + The Basic tier supports autoscaling, allowing the application to scale and handle user load at peak times.
- The Azure Function with the Consumption tier is cost-effective:
   + We only pay for the actual execution time and resource consumption of your functions. We are not billed for idle time, making it a cost-effective option for sporadic or low-traffic workloads.
   + When used with Azure Service Bus, it allows sending notifications to attendees asynchronously, preventing HTTP timeouts in the traditional architecture.
- In the new Azure architecture, the impact comes from the Azure Postgres Database. Although it uses the General Purpose tier with 100 GiB storage, it has a significantly higher cost compared to other services.