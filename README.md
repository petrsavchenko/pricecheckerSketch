# Pricechecker App - Design Sketch

Pricechecker app is a web application that monitors products on online shops and alerts users when prices reach user acceptable level, helping them find the perfect moment to buy.

There is a list of features:
* register users
* allows users to setup products to be monitored and define acceptance criteria on prices
* monitor price of the product in online shop
* notify users if prices meet their criteria
* display history of changing prices of the products

Main priorities of the service are availability and reliability. Scalability and extensibility are crucial as the app is going to monitor a lot of products and the number of online shops will be growing and their structure (API, markup) will be changing.

High level design:
1. In front of API Gateway with authentication, throttling and CORS setup. API Gateway will have basic CRUD operations and integrated with DynamoDb. Validation and correctness of data handled by JSON Schema Validators. It would be the most cost optimized solution as no lambda is triggered. 
2. Once record is inserted to DynamoDb, this will invoke auto scalable and highly available Lambda functions to handle crawlering. Results of the crawling (contains price and productId) will be dumped as the event to SQS to be processed on step 4. If not, then the function will schedule the next invocation. To do so it will send a message to SQS with a message timer (in case if customer wants to customize frequency of price checks) or call step functions with repeated lambda invocation after the timer expires.
3. Results of the crawlering will be pushed to DynamoDb so the record will be updated with the latest price. DynamoDb was selected as it will be the most cost effective solution and easily scale up horizontally as well as good options in case of disaster recovery.
4. SQS Events from step 2 are being processed by the second lambda fleet on the price update which triggers another lambda. When it checks if any user should be notified about the change. Notifications are being sent through SNS.
5. Frontend assets will be stored in S3 bucket and served via cloudfront.

There is a diagram illustrating proposed design below. You also can view and interact with [the diagram here](https://app.cloudcraft.co/view/9ffe2049-7e67-4ba3-b843-fab5766af1c4?key=9KUDq4HNSBZO0PKrXnNm2g):

![design diagram](/dia.png)
