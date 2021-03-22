# Design a Data Getter - DigitalClues

## 1. Use cases and assumptions

### Use cases
* User update configuration and start the process
    * User update 5 publicly available REST API services in configuration file
    * User update the "n" number in configuration file
        * bottleneck 1: n is equal to concurrent threads in the Request Perform Service and each thread take up to 500KB virtual memory
        * bottleneck 2: MongoDB can handle up to 30K requests/second
    * User start process
    
### State assumptions
* all the REST API services are stable and fast responsible - up to 333ms per request
* .NET Core hits up to 300K requests per second (2019)
* RabbitMQ hits up to 1M messages per second (2014)
* .NET Core hits up to 32K concurrent threads
* MongoDb hits up to 30K requests per second
* each request url length (rest api url) can be up to 500 bytes
* response volume is JSON type and can be up to 500 bytes
* "n" number (tasks in parallel) can be between 1 and up to server memory limit - 
for each 1Gb available memory we could create ~ 1Gb / (500 + 500 + 500K) = ~2K parallel threads but total number of threads can not be greater then 32K parallel threads (.NET limit)

### Calculate usage

* Volume size data block per one task/thread
    * 500B max per rest api url
    * 500B max per JSON data response
    * total = ~1KB
* Stack size of a thread running on an x64 server is 512K

## 2. High level design

![Imgur](https://github.com/simonbor/DigitalClues/hight-level-design.png)

## 3. Design core components

### Use case: User update configuration and start the process
* User update Request Generation Service configuration
    * User update 5 publicly available REST API urls
* User update Request Perform Service configuration
    * User update "n" number configuration (tasks in parallel)
* User start processes
    * User start the Request Perform Service
        * first thread created
        * first thread get API url and priority
        * first thread request API
        * first thread wait for response
        * first thread obtain response and HTTP Status
        * first thread creates first message in Message Queue
        * Queue Consumer receive first message and create a new document in Mongo (id, url, priority, timestamp, http status, response)
        > second thread, third thread, etc.

## 4. Scale the design (BONUS)

![Imgur](https://github.com/simonbor/DigitalClues/scale-level-design.png)

