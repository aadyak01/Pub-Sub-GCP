# Google Cloud Pub/Sub SpringBoot Publisher Tutorial
Welcome to the Google Cloud Pub/Sub Java Publisher Tutorial! This tutorial will guide you through the process of setting up a Java application to publish messages to Google Cloud Pub/Sub topics using the Google Cloud Pub/Sub Java client library.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Setting Up Google Cloud Environment](#setting-up-google-cloud-environment)
3. [Setting Up Your SpringBoot Project](#setting-up-your-SpringBoot-project)
4. [Writing the Publisher Code](#writing-the-publisher-code)
5. [Handling Errors and Exceptions](#handling-errors-and-exceptions)
6. [Testing Your Publisher](#testing-your-publisher)
7. [Additional Features and Best Practices](#additional-features-and-best-practices)
8. [Conclusion](#conclusion)

## Prerequisites
- Google Cloud account with appropriate permissions.
- Java Development Kit (JDK) installed.
- [Maven](https://maven.apache.org/) for managing dependencies.

## Setting Up Google Cloud Environment
1. Authenticate your Google Cloud account.
2. Create a Google Cloud Pub/Sub topic for publishing messages.

Detailed instructions can be found in [Google Cloud Documentation](https://cloud.google.com/pubsub/docs).

## Setting Up Your SpringBoot Project
1. Create a new SpringBoot project or use an existing one.
2. Add the Google Cloud Pub/Sub Java client library as a dependency in your project's `pom.xml`:

   ```xml
   <dependency>
       <groupId>com.google.cloud</groupId>
       <artifactId>google-cloud-pubsub</artifactId>
       <version>INSERT_VERSION_HERE</version>
   </dependency>


To create a publisher for Google Cloud Pub/Sub in Java, you'll need to use the Google Cloud Pub/Sub Java client library. Here are the steps to publish data to a topic in Google Cloud Pub/Sub using Java:

1. Set Up Google Cloud SDK and Project:

Before you start coding, ensure you have Google Cloud SDK installed and configured with a Google Cloud project. You can follow Google's documentation for setting up the SDK and creating a project: [https://cloud.google.com/pubsub/docs](url)

2. Add Dependencies:

   Add the Google Cloud Pub/Sub client library to your Java project. You can include it in your Maven or Gradle project as follows:

   Maven:

   ```xml
   <dependencies>
       <dependency>
           <groupId>com.google.cloud</groupId>
           <artifactId>google-cloud-pubsub</artifactId>
           <version>1.125.2</version> <!-- Use the latest version -->
       </dependency>
   </dependencies>
   ```

   Gradle:

   ```groovy
   implementation 'com.google.cloud:google-cloud-pubsub:1.125.2' // Use the latest version
   ```

## Writing the Publisher Code:

   Here's an example of a Java program that publishes data to a Google Cloud Pub/Sub topic:

```java
import com.google.api.gax.core.FixedCredentialsProvider;
import com.google.auth.oauth2.ServiceAccountCredentials;
import com.google.cloud.pubsub.v1.Publisher;
import com.google.cloud.pubsub.v1.PublisherInterface;
import com.google.protobuf.ByteString;
import com.google.pubsub.v1.PubsubMessage;
import com.google.pubsub.v1.TopicName;

import java.nio.charset.StandardCharsets;
import java.nio.file.Paths;

@Service
public class PubsubService {

    // Read value of projectId and topicId by specifying it in application.prop

    @Value("${google.cloud.pubsub.project-id}")
    private String projectId;

    @Value("${google.cloud.pubsub.topic-id}")
    private String topicId;
    
    private static final Logger logger = Logger.getLogger(PubsubService.class.getName());
	 
    public String publishMessage(Object dataObject) {
    	
    	logger.info("----------- Inside Pub Sub Service ----------- ");
    	logger.info(" projectId: "+projectId);
    	logger.info(" topicId: "+topicId);

     //String jsonKeyPath = "path-to-your-service-account-key.json";

/**However keeping your service-account-key.json file anywhere in the project is not recommended but providing jsonKeyPath to your local won't work on
dev, qa or any other env.Here is a way to make it compatible on every environment, for this keep service-account-key.json in your
application's resource folder**/

		InputStream inputStream = PubsubService.class.getResourceAsStream("/service-account-key.json");
		
		try {
			logger.info("Inside try pub sub logic");
			
          //Read the service account JSON key from the file
            GoogleCredentials credentials = GoogleCredentials.fromStream(inputStream);

          // Create a CredentialsProvider using the credentials
	           CredentialsProvider credentialsProvider = FixedCredentialsProvider.create(credentials);

          // Set up the topic name
              ProjectTopicName topicName = ProjectTopicName.of(projectId, topicId);
              logger.info("Creating a Pub/Sub publisher");
        	// Creating a Pub/Sub publisher
        	  Publisher publisher = Publisher.newBuilder(topicName)
        		  .setCredentialsProvider(credentialsProvider)
        		  .build();
        	
        	//Mapping Object as Json String
        	ObjectMapper mapper = new ObjectMapper(); 
        	String jsonString=mapper.writeValueAsString(dataObject);
        	logger.info("Mapped Object to JSON string:"+jsonString );
        	
          // Publish the JSON string as a PubsubMessage
        	logger.info("Publish the JSON string as a PubsubMessage");
            PubsubMessage pubsubMessage = PubsubMessage.newBuilder()
            .setData(ByteString.copyFromUtf8(jsonString))
            .build();
            
             publisher.publish(pubsubMessage);
             
             logger.info("Shutting down the publisher");
             publisher.shutdown();
             
             logger.info("Message Content: " + pubsubMessage.getData().toStringUtf8());
                    
            
        } catch (Exception e) {
            e.printStackTrace();
            return "Error publishing message: " + e.getMessage();
        }
        return "Message published successfully.";
	}
 
	
}
```

4. Replace the placeholders in the code with your own Google Cloud project ID, topic ID, and the path to your service account JSON key file/service account JSON key kept in app resource folder. Define all the placeholders in your application.prop like below:

### ==========================================
### Pub/Sub Configurations in application.prop
### =========================================
 ```
google.cloud.pubsub.project-id= your-project-id
google.cloud.pubsub.topic-id= your-topic-id
 ```

5. Autowire PubsubService in your controller and you can publish data like below:
    @Autowired
	  PubsubService pubsubService;

    pubsubService.publishMessage(result);

Make sure you have the necessary permissions and authentication set up to access the Google Cloud Pub/Sub service with the provided service account key.

## Handling Errors and Exceptions
Implement error handling to gracefully manage any issues during publishing.
**Try-Catch Blocks:**
To handle exceptions in your code, you use try-catch blocks.The code that might throw an exception is placed inside the try block. In the catch block, you specify what should happen if an exception of a specific type occurs. Multiple catch blocks can be used to handle different types of exceptions. (Refer above code snippet for more clarity)

## Testing Your Publisher
Verify that your Java application can successfully publish messages to the Pub/Sub topic. You can do that by pulling the published message via subscriber's PULL option on GCP. 

## Additional Features and Best Practices
1. Explore advanced features like message batching and custom attributes.
2. Follow best practices for efficient message publishing.
   
## Conclusion
Congratulations! You've successfully set up a Google Cloud Pub/Sub publisher in your SprinBoot application. Feel free to customize this code to suit your specific use case.
