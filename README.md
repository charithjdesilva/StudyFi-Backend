# StudyFi-Backend
Backend for the Study-Fi Project

## Project Description
StudyFi is a social media platform designed to help students connect, collaborate, and share resources. The backend is a microservices-based application built with Spring Boot, providing various functionalities such as user management, group creation and management, content and news sharing, chat features, and notifications.
## Technologies Used
- **Spring Boot:** Framework for building the microservices.
- **Spring Cloud:** For service discovery (Eureka) and API Gateway.
- **Spring Data JPA:** For database interaction.
- **MySQL:** Relational database.
- **Maven:** Dependency management and build automation.
- **Lombok:** To reduce boilerplate code.
- **Cloudinary:** Used for file hosting, supporting various file types including pdf, doc, pptx, jpg, png, and webp.
- **Google SMTP (or similar):** Configured for sending email notifications, such as for user verification and password resets.
- **WebClient:** Utilized for making **asynchronous** HTTP requests between microservices, enabling interservice communication. While synchronous usage is possible, the asynchronous nature of WebClient is leveraged for better performance and scalability in this microservices architecture.
- **Eureka Server:** For service discovery.
- **Spring Cloud Gateway:** For API Gateway.
## Project Setup
1.  **Clone the repository:**
```
bash
git clone https://github.com/studyfi/StudyFi-Backend.git
```
2.  **Database Setup:**
    *   Install MySQL if you don't have it already.
    *   Create a database for the project. You can use the MySQL command line or a GUI tool like MySQL Workbench.
    *   Update the database connection properties in the `application.properties` file for each microservice (e.g., `contentandnews/src/main/resources/application.properties`, `userandgroup/src/main/resources/application.properties`). You will need to configure the database URL, username, and password.

3.  **Environment Variables:**
    *   Configure the necessary environment variables for each microservice. This may include:
        *   Cloudinary credentials (cloud name, API key, API secret)
        *   SMTP server details (host, port, username, password)
        *   Any other service-specific configurations.

4.  **Build the project:**
    *   Navigate to the project's root directory in your terminal.
    *   Build the project using Maven:
```
bash
mvn clean install
```
## Project Folder Structure

The project is organized into several directories, with each microservice residing in its own folder at the root level:

*   `apigateway`: Contains the code for the API Gateway service.
*   `discoveryserver`: Contains the code for the Discovery Server (Eureka).
*   `userandgroup`: Contains the code for the User and Group service.
*   `contentandnews`: Contains the code for the Content and News service.
*   `notification`: Contains the code for the Notification service.

Each service folder typically follows a standard Spring Boot project structure, including directories like `src/main/java` for source code and `src/main/resources` for configuration and static files.


## API Gateway Service

### Project Description
The `apigateway` service, powered by **Spring Cloud Gateway**, serves as the crucial entry point for all client interactions with the StudyFi backend. It eliminates the need for clients to know the specific addresses of individual microservices by providing a unified API.

### Architectural Overview
The API Gateway service is built using **Spring Cloud Gateway**. Its core architecture revolves around **routing configurations**, dynamically determined based on **service discovery** from the Eureka Server. It acts as a reverse proxy, directing incoming client requests to the appropriate downstream microservices. Key components involved include:

*   **Predicates:** Used to match incoming HTTP requests based on various criteria (e.g., path, headers, query parameters).
*   **Filters:** Allow for modifying the request or response, or performing pre/post-processing logic (e.g., authentication, logging, rate limiting).

The Gateway retrieves the available instances of a target service from Eureka and forwards the request, potentially applying load balancing across multiple instances. It commonly runs on port **8080** by default.

### Functionalities
The API Gateway service acts as the single entry point for all client requests to the StudyFi backend. It is responsible for routing incoming requests to the appropriate microservices. Key functionalities include:

*   **Request Routing:** Routing incoming client requests to the appropriate downstream microservices based on predefined routes typically defined in the `application.properties` or `application.yml` file.
*   **Service Discovery (Eureka Server):** Acts as a Eureka Client and registers with the Eureka Server to discover the network locations (IP address and port) of all registered microservice instances.
*   **Cross-Cutting Concerns:** Handling cross-cutting concerns that apply to multiple services, such as:
    *   **Authentication and Authorization:** Verifying user identity and permissions before requests reach the microservices.
    *   **Rate Limiting:** Protecting backend services from excessive requests.
    *   **Logging:** Centralizing request logging.
*   **Load Balancing:** Can potentially handle load balancing across multiple instances of a microservice.

### Routing Configuration Example
The API Gateway's routing is configured in the `application.properties` file. Here is an example of how requests are routed to the `user-service`:
```
properties
spring.cloud.gateway.routes.0.id=user-service
spring.cloud.gateway.routes.0.uri=lb://user-service
spring.cloud.gateway.routes.0.predicates[0]=Path=/api/users/**
```
### Technologies and Components
*   This service is built using **Spring Cloud Gateway**.
*   Key dependencies are listed in the `apigateway/pom.xml` file, including `spring-cloud-starter-gateway` and dependencies for service discovery (Eureka Client).


## Discovery Server
The `discoveryserver` service acts as the **Eureka Server**, which is central to the StudyFi microservices architecture. Its fundamental role is to facilitate **service registration and discovery**.

### Architectural Overview
The Discovery Server's architecture is based on the **Eureka Server** pattern. It functions as a central registry where each microservice in the StudyFi backend registers its instance information (like network location and service ID) upon startup. This registry is then available for clients, primarily the **API Gateway** and potentially other microservices themselves, to query and discover the network addresses of required services. This dynamic discovery mechanism is essential for the microservices to locate and communicate with each other without needing hardcoded configurations.

It serves as the registry where each microservice (like `userandgroup`, `contentandnews`, and `notification`) registers itself upon startup. Other services, including the API Gateway and potentially the microservices themselves for inter-service communication, can query the Discovery Server to obtain the network locations (IP addresses and ports) of the registered service instances. This dynamic discovery allows services to find and communicate with each other without needing hardcoded addresses.

While microservices can communicate directly with each other after discovering their locations via the Eureka Server, the **API Gateway remains the primary entry point for all external client requests**, handling routing and cross-cutting concerns.

The Discovery Server commonly runs on port **8761** by default.

### Functionalities
*   **Service Registration:** Microservices register their instance information (host, port, service ID) with the Eureka Server upon startup.
*   **Service Discovery:** Clients (like the API Gateway or other microservices) can query the Eureka Server to get a list of available instances for a specific service ID.
*   **Heartbeats:** Registered services send periodic heartbeats to the Eureka Server to indicate they are still alive and operational. The server will deregister instances that stop sending heartbeats.

### Technologies and Components
*   This service is built using **Spring Cloud Netflix Eureka Server**.
*   Key dependencies are listed in the `discoveryserver/pom.xml` file, including `spring-cloud-starter-netflix-eureka-server`.

### Configuration Details
The `discoveryserver/src/main/resources/application.properties` file contains specific configurations for the Eureka Server. Key properties include:

*   `eureka.instance.hostname=localhost`: Explicitly sets the hostname for the Eureka server instance to `localhost`. Typically used in development or testing environments. In production, this would usually be a proper hostname or IP address.
*   `eureka.client.register-with-eureka=false`: Prevents the Eureka server from registering itself as a client in its own service registry. Standard practice for a Eureka server.
*   `eureka.client.fetch-registry=false`: Instructs the Eureka server not to fetch the service registry from other Eureka servers. Relevant in a single-instance Eureka setup. In a clustered environment, this would be set to `true`.

### How Services Connect and Communicate

Microservices connect to the Discovery Server by including the `spring-cloud-starter-netflix-eureka-client` dependency and configuring the Discovery Server's address in their `application.properties` or `application.yml`.

Upon startup, they register themselves with the Discovery Server. Other services, acting as Eureka Clients, can then query the Discovery Server to obtain the locations of the required service instances.

This enables direct interservice communication using clients like `WebClient` or `Feign Client` that are aware of the Discovery Server. While direct communication is possible internally, the API Gateway remains the single entry point for all external client requests.


## User and Group Service

The `userandgroup` service is responsible for managing user accounts and groups within the StudyFi platform. It handles all operations related to user profiles, authentication, and the creation and management of study groups.

### Architectural Overview

This service is built with **Spring Boot** and leverages **Spring Data JPA** for seamless interaction with the **MySQL database**, handling all data persistence for users and groups. It integrates with **Cloudinary** for efficient file storage, specifically for user profile and cover images, as well as group images. For communication with other microservices, particularly the Notification service for sending emails, it utilizes **WebClient** for asynchronous interactions.

### Functionalities

*   **User Management:**
    *   User Registration and Account Creation
    *   User Login and Authentication
    *   User Profile Management (viewing and updating profile information, including managing **profile images (`profile_image_url`)** and **cover images (`cover_image_url`)**)
    *   Email Verification (for new user accounts)
    *   Password Reset functionality
*   **File Uploads (User and Group Images):**
    *   Handling file uploads for user profile and cover images using Cloudinary.
    *   Handling file uploads for group images using Cloudinary.
*   **Group Management:**
    *   Creating new study groups
    *   Managing group members (listing, potentially adding/removing)

### Database Structures

The User and Group Service interacts with the MySQL database, primarily managing the following tables:

**`user` Table**

| Column Name               | Data Type      | Description                                                 |
|---------------------------|----------------|-------------------------------------------------------------|
| `id`                      | BIGINT         | Primary Key                                                 |
| `name`                    | VARCHAR        |                                                             |
| `email`                   | VARCHAR        | Unique                                                      |
| `password`                | VARCHAR        |                                                             |
| `phoneContact`            | VARCHAR        |                                                             |
| `birthDate`               | DATE           |                                                             |
| `country`                 | VARCHAR        |                                                             |
| `aboutMe`                 | TEXT           |                                                             |
| `currentAddress`          | VARCHAR        |                                                             |
| `profile_image_url`       | VARCHAR        | URL to the user's profile image stored on Cloudinary        |
| `cover_image_url`         | VARCHAR        | URL to the user's cover image stored on Cloudinary          |
| `verification_code`       | VARCHAR        | Code for password reset                                     |
| `verification_code_expiry`| DATETIME       | Expiry time for the password reset code                     |
| `email_verification_code` | VARCHAR        | Code for email verification                                 |
| `email_verification_code_expiry`| DATETIME   | Expiry time for the email verification code                 |
| `emailVerified`           | BOOLEAN        | Indicates if the user's email has been verified             |

**`study_group` Table**

| Column Name   | Data Type      | Description                             |
|---------------|----------------|-------------------------------------------------|
| `id`          | BIGINT         | Primary Key                                     |
| `name`        | VARCHAR        |                                                 |
| `description` | TEXT           |                                                 |
| `image_url`   | VARCHAR        | URL to the group's image stored on Cloudinary |

**`user_group` Table**

| Column Name   | Data Type      | Description                     |
|---------------|----------------|-------------------------------------|
| `user_id`     | BIGINT         | Foreign Key referencing the `user` table |
| `group_id`    | BIGINT         | Foreign Key referencing the `study_group` table |

<img src="img.png" align="center" alt="user and group db" width="600"/>

### Technologies and Components

*   **Spring Data JPA:** Used for interacting with the MySQL database to persist user and group data.
*   **Cloudinary:** Integrated for handling the storage of user profile images (`profile_image_url`, `cover_image_url`) and group images (`image_url`). Configuration details for Cloudinary are expected to be in the `application.properties` or a dedicated configuration file.
*   **Email Functionality:** The presence of fields like `email_verification_code` and `emailVerified` in the User entity indicates email-related features (like verification and password reset). Email sending is handled through SMTP configuration (likely in `application.properties` with a configuration file).
*   **WebClient:** Explicitly utilized for **asynchronous** interservice communication, including making calls to other services for tasks such as sending notifications.

### Custom Exceptions

The service defines custom exceptions within the `com.studyfi.userandgroup.user.exception` package to handle specific error scenarios related to user operations:
*   `EmailNotVerifiedException`: Thrown when an action requires a verified email but the user's email is not verified.
*   `EmailVerificationCodeInvalidException`: Thrown when an provided email verification code is invalid.
*   `InvalidVerificationCodeException`: Thrown when a verification code (potentially for password reset or email verification) is invalid.

### Code Expiration

The `verification_code_expiry` and `email_verification_code_expiry` fields in the `user` table are used to manage the validity periods for password reset and email verification codes, respectively. This ensures that these codes expire after a certain time, enhancing security by preventing indefinite use of a code. The service is responsible for setting these expiration times when codes are generated and checking against them during the verification or reset process.

The service defines custom exceptions within the `com.studyfi.userandgroup.user.exception` package to handle specific error scenarios related to user operations:
*   `EmailNotVerifiedException`: Thrown when an action requires a verified email but the user's email is not verified.
*   `EmailVerificationCodeInvalidException`: Thrown when an provided email verification code is invalid.
*   `InvalidVerificationCodeException`: Thrown when a verification code (potentially for password reset or email verification) is invalid.

The User and Group Service interacts with other microservices in the following ways:

### Interaction with Other Services

*   **Outbound:** It can call the Notification service (via WebClient) to send user or group related notifications.
*   **Inbound:** Other microservices (e.g., Content and News service) may call the User and Group service to retrieve user or group information (like names, profile pictures) to display alongside content or group details.

## Content and News Service

### Project Description
The `contentandnews` service is responsible for managing all content and news within the StudyFi platform. This includes handling news articles, user-generated posts, and interactions such as likes and comments. It serves as the central hub for information dissemination and user engagement through content.

### Architectural Overview
This service is built with **Spring Boot** and utilizes **Spring Data JPA** for interacting with the MySQL database to persist data related to content, news, posts, likes, and comments. It integrates with **Cloudinary** for handling file uploads associated with posts, such as images or documents. For interservice communication, it employs **WebClient**, for example, to fetch user information (like names and profile pictures) from the User and Group service to display alongside content and comments.

### Functionalities
*   **Content and News Management:**
    *   Creating, viewing, updating, and deleting news articles and user posts.
    *   Retrieving lists of content (e.g., recent news, posts within a group).
*   **Interaction Management:**
    *   Handling likes on news articles and posts.
    *   Managing comments on news articles and posts (creating, viewing, potentially deleting).
*   **File Uploads:**
    *   Handling file uploads associated with posts (images, documents, etc.) via Cloudinary.

### Technologies and Components
*   **Spring Boot:** Framework for building the microservice.
*   **Spring Data JPA:** For database interaction.
*   **MySQL:** Relational database for storing content, news, likes, and comments data.
*   **Cloudinary:** Integrated for handling file storage for post attachments.
*   **WebClient:** Used for asynchronous communication with other microservices, such as the User and Group service.

### Database Structures

The Content and News Service interacts with the MySQL database, managing the following tables for various types of content, posts, likes, and comments:

**`content` Table**

| Column Name   | Data Type   | Description                                                                    |
|---------------|-------------|--------------------------------------------------------------------------------|
| `id`          | INT         | Primary Key, auto-generated                                                    |
| `title`       | VARCHAR     | Title of the content                                                           |
| `content`     | TEXT        | Content text (optional, can be used for description)                           |
| `author`      | VARCHAR     | Author of the content (user who uploaded) - stored as a String               |
| `groupIds`    | (See note)  | List of Group IDs where the content is shared                                  |
| `createdAt`   | DATE        | Timestamp when the content was created/posted                                  |
| `fileURL`     | VARCHAR     | URL to the file stored on Cloudinary or other file storage                   |

**`news` Table**

| Column Name   | Data Type   | Description                                                                 |
|---------------|-------------|-----------------------------------------------------------------------------|
| `id`          | INT         | Primary Key, auto-generated                                                 |
| `headline`    | VARCHAR     | Title or headline of the news                                               |
| `content`     | TEXT        | Body of the news (text or URL)                                              |
| `author`      | VARCHAR     | Author of the news (user who posted) - stored as a String                   |
| `groupIds`    | (See note)  | List of Group IDs where the news is shared                                  |
| `createdAt`   | DATE        | Timestamp of content creation                                               |
| `imageUrl`    | VARCHAR     | URL of the image associated with the news (optional) stored on Cloudinary   |

**Note on `groupIds`:**

For both `content` and `news` tables, the `groupIds` field is handled using `@ElementCollection`. This results in separate join tables (named `content_groupIds` and `news_groupIds`) to store the relationship between the content/news item and the associated group IDs. These join tables usually have columns referencing the content/news ID and the group ID.

**`post` Table**

| Column Name   | Data Type     | Description                                            |
|---------------|---------------|--------------------------------------------------------|
| `postId`      | INT           | Primary Key, auto-generated                            |
| `groupId`     | INT           | The ID of the group the post belongs to              |
| `userId`      | INT           | The ID of the user who created the post                |
| `content`     | TEXT          | The content of the post                                |
| `timestamp`   | DATETIME      | The time the post was created                          |

**`post_like` Table**

| Column Name   | Data Type  | Description                                     |
|---------------|------------|-------------------------------------------------|
| `likeId`      | INT        | Primary Key, auto-generated                     |
| `userId`      | INT        | Foreign Key referencing the `user` table        |
| `postId`      | INT        | Foreign Key referencing the `post` table        |

*Unique constraint:* `uq_user_post_like` ensures a user can like a post only once.

**`comment` Table**

| Column Name   | Data Type  | Description                                     |
|---------------|------------|-------------------------------------------------|
| `commentId`   | INT        | Primary Key, auto-generated                     |
| `postId`      | INT        | Foreign Key referencing the `post` table        |
| `userId`      | INT        | Foreign Key referencing the `user` table        |
| `content`     | TEXT       | The content of the comment                      |
| `timestamp`   | DATETIME   | Timestamp of when the comment was created       |

<img src="img_1.png" align="center" alt="content and news db" width="600"/>

### Service Interactions
The Content and News Service interacts with other microservices in the following ways:

*   **Outbound:** It calls the User and Group service (via WebClient) to retrieve author information (name, profile picture) to display with news articles and posts. It also interacts with the Notification service to trigger notifications for new comments or likes.
*   **Inbound:** The API Gateway routes client requests related to content and news to this service. **Additionally**, other services query this service for content after discovering its location via the Discovery Server, which allows for more efficient direct communication.

## Notification Service

### Project Description
The `notification` service is dedicated to handling and sending notifications within the StudyFi platform. It acts as a centralized component for triggering various types of notifications in response to events from other services, ensuring users are informed about relevant activities such as new content, group updates, or account-related events.

### Architectural Overview
Built with **Spring Boot**, the Notification Service is designed to receive notification requests from other microservices. These requests likely contain the necessary information to compose and send a notification (e.g., recipient user ID, message content, notification type). It leverages technologies for sending notifications, such as an SMTP client for emails. Its architecture is focused on processing incoming notification requests efficiently and interacting with external notification delivery systems.

### Functionalities
*   Receiving notification requests from other services.
*   Composing notification messages based on received data.
*   Sending notifications, primarily via email (based on the project's overall technologies).
*   Handling different types of notifications (e.g., user verification, password reset, new comment, group activity).

### Technologies and Components
*   **Spring Boot:** Framework for building the microservice.
*   **WebClient:** Potentially used for receiving requests from other services or interacting with external notification APIs.
*   **SMTP Client (e.g., JavaMailSender):** Used for sending email notifications.
*   **Maven:** Dependency management and build automation.

### Service Interactions
The Notification Service primarily has **inbound** interactions, receiving calls from other services to trigger notifications:
*   **Inbound:** Receives requests from services like the User and Group Service (for email verification, password reset) and the Content and News Service (for new comments, likes, etc.) to send specific notifications to users.
*   **Outbound:** May interact with external services or APIs for delivering notifications (e.g., an external email service via SMTP, or future integrations with push notification services).

### Database Structures
The Notification Service interacts with a database to store notification information. The primary table is `notifications`:

**`notifications` Table**

| Column Name     | Data Type     | Description                       |
|-----------------|---------------|-----------------------------------|
| `notification_id` | INT           | Primary Key, auto-generated       |
| `message`         | VARCHAR       | The notification message          |
| `user_id`         | INT           | The ID of the recipient user      |
| `is_read`         | BOOLEAN       | Indicates if the notification is read (defaults to false) |
| `timestamp`       | DATETIME      | The time the notification was created |
| `group_id`        | INT           | The ID of the associated group (if any) |
| `group_name`      | VARCHAR       | The name of the associated group (if any) |


