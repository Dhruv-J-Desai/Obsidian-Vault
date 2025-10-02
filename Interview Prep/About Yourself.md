- Hi, I’m Dhruv Desai. 
- I have over 5 years of experience as a Java Full Stack Developer and am currently working as a Senior Software Engineer at Reactivestax since past one year.
--------
- My core expertise is in backend development with Java, Spring Boot, and microservices, along with building event-driven architectures using Kafka. 
- I have hands-on experience with testing frameworks like JUnit, Mockito, and WebMvc for both integration and mock testing. 
- I’ve worked extensively with relational databases such as PostgreSQL, Oracle, and SQL Server, as well as NoSQL databases like MongoDB, and I use Redis for caching. 
- On the deployment side, I’m experienced with Docker, Kubernetes, OpenShift, and CI/CD pipelines. 
- I also have experience with front-end technologies like React and Angular.
-------------
In my most recent project, I’ve been working on a trading application that processes large trade files. The application first chunks the incoming file into multiple parts and uses `ExecutorService` to spawn multiple threads, with each thread processing one chunk. Each worker thread saves the processed rows into a `rawTradeTable`.

We then validate each trade by checking its CUSIP against a separate CUSIP table. If the CUSIP exists, the trade is marked as valid and moved to a `validTrades` table. From there, a Kafka producer publishes each valid trade to a Kafka topic. On the consumer side, a Kafka consumer continuously polls messages, and upon receiving them, we perform a batch upsert into MongoDB based on `userId` and `tradeId`.


Requirements gathering > BSA > 
Swagger
Security...Basic Auth/ Form Login/ TLS/  OAuth (along with its flows)
Request/ Response structure
And then distributed it to different teams (frontend, backend, testing)
**codegen**

