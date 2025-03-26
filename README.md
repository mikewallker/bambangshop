# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. In the Observer pattern, an interface (or trait in Rust) is typically used to allow multiple types of subscribers to implement different behaviors when receiving updates. However, in the BambangShop case all subscribers behave the same way (e.g., just storing a URL for notifications), hence a single Subscriber struct is enough. However, if we plan to have different types of subscribers (e.g., different notification mechanisms like email, SMS, or webhook), then using a trait would be better to enforce a common contract.

2. Since id (in Program) and url (in Subscriber) must be unique, using a Vec<Subscriber> means every operation (add, delete, list_all) would require iterating over all elements to check for duplicates. This is inefficient, especially if the number of subscribers grows large. DashMap, as a concurrent HashMap, allows fast lookups by key (url), making it much more efficient for checking uniqueness and retrieving subscribers.Thus, DashMap is necessary to avoid performance issues and unnecessary iterations.

3. The Singleton pattern ensures only one instance of SUBSCRIBERS exists globally, which prevents accidental duplication. However, Singleton alone does not provide thread safety for concurrent reads and writes. Rust enforces strict concurrency rules, and DashMap is already optimized for safe concurrent access. If we implement a Singleton pattern without DashMap, we would still need to use locking mechanisms (e.g., Mutex<HashMap> or RwLock<HashMap>) to ensure thread safety, which can introduce performance bottlenecks. Thus, DashMap remains the better choice for this case since it provides both thread safety and efficient concurrent access without explicit locking.

#### Reflection Publisher-2
1.  Although the traditional Model-View-Controller (MVC) pattern treats the Model as the component that handles both data storage and business logic, modern software design principles advocate for a separation of concerns to improve scalability, maintainability, and testability. This is why we introduce Service and Repository layers. The Repository layer acts as an abstraction over the database or data storage. It provides structured access to data without exposing database implementation details to higher layers. By doing this, we achieve loose coupling, making it easier to switch databases or use different data sources. The Service layer encapsulates business logic and ensures that models interact in a cohesive manner. It prevents fat controllers by keeping business logic separate from request-handling logic. This makes it easier to unit test business logic independently.

2. If we don't separate Service and Repository layers, the Model would have to query the database (directly interacting with raw SQL, ORM, or other DB mechanisms), process business logic (e.g., handling subscriptions, notifications), and will expose data to controllers (handling serialization, transformations). This leads to:

    Tightly coupled code → The Model is responsible for too many things.

    Reduced reusability → Other parts of the system cannot reuse business logic independently.

    Difficult testing → Unit testing becomes harder since the Model directly depends on external data sources.

    Scalability issues → Adding new business logic requires modifying Models, increasing the risk of breaking changes.

    If we only use the Model, the interactions between Program, Subscriber, and Notification will lead to tightly coupled logic.

    - Scenario Without Service and Repository

    Program might need to directly query Subscriber and Notification, leading to duplicate code in different places. If Notification has logic related to Subscriber, the Subscriber model might become bloated with unrelated notification logic. Any change in Subscriber (like a schema update) could break Notification logic inside the Model.

    - Scenario With Service and Repository

    ProgramService interacts with SubscriberService and NotificationService indirectly, reducing interdependencies. Each model remains focused on its purpose (data structure) while services handle orchestration. The system is easier to test, maintain, and extend.

3. Yes, I have explored Postman, and it is extremely helpful for API testing. It allows me to simulate requests without needing a frontend or manual execution via CLI tools like curl. I can send GET, POST, PUT, DELETE requests with different payloads. It helps validate request/response formats before integrating with the frontend. One of the feature I find helpful for future project is Monitor API Performance that Helps track API uptime and response time.

#### Reflection Publisher-3
1. In this implementation, the code uses a Push model of the Observer Pattern. This is evident in the notify() method, where the publisher actively creates a notification payload and pushes it to all subscribers using thread::spawn(). The update() method in the Subscriber implementation demonstrates this push mechanism, where the notification is sent directly to each subscriber with pre-populated data.

2. Advantages of Pull Model:

    Reduced network traffic, as subscribers only request updates when they need them. Lower computational overhead for the publisher. More control for subscribers over when they fetch updates. Potentially more efficient for subscribers with intermittent update needs

    Disadvantages for this specific use case:

    Increased complexity in tracking state changes. Potential for missed updates if subscribers don't poll frequently. More responsibility placed on subscribers to manage update retrieval. Higher latency in receiving notifications. Increased load on the system due to frequent polling

    Specific to this product/notification system, a pull model would require Maintaining a more complex state tracking mechanism, Implementing a polling mechanism for subscribers and Potentially missing time-sensitive notifications like product creation, deletion, or promotions

3. Without multi-threading (removing thread::spawn()), the notification process would become:

- Synchronous: Each subscriber would be notified sequentially
- Blocking: The notification process would wait for each subscriber to complete before moving to the next
- Potential performance bottlenecks: If one subscriber is slow to respond, it would delay notifications to other subscribers. The entire notification process would take much longer. Also there is a Risk of timeout or system unresponsiveness if a subscriber takes too long to process
