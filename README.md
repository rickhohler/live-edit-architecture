# Live Edit Architecture

The scenario involves a user utilizing an app to view their customer account data. It is essential that any updates made to the customer account through other apps are promptly reflected to the user while they are accessing their data.

## ACME

In our illustrative scenario, let's consider ACME, a company that serves customers who make purchases of widgets.

## Customer Account

A customer account consists of the following information, represented in JSON format:

```json
{
  "id": "46E216CA-9413-4E7E-B5C7-DFAA5AEADF9E",    // UUID generated id
  "name": "John Smith",
  "address": {
    "street1": "101 Main Street",
    "street2": null,
    "city": "Washington DC",
    "state": "DC",
    "zip": "20001"
}
```

## Proposed Design

In order to establish efficient communication, we will adopt the publish/subscribe (pub/sub) pattern, which allows us to promptly inform the user about any modifications made to their account data.

Our design will be hosted on AWS, leveraging its robust infrastructure.

### Participants / Components

* **Customer App** - ACME has developed an app that empowers customers to effortlessly access and manage their account data, enabling them to view and update it with ease.
* **Representive App** - ACME help desk representatives utilize the Representative App to efficiently modify customer data on behalf of the users whenever they contact the company via phone or chat.
* **ACME Server** - An AWS-hosted system that leverages the power of Cognito and Simple Notification Service (SNS) products to enhance its functionality and deliver efficient services.
* **Live Topic** - An AWS SNS topic to streamline communication and facilitate seamless interaction whenever changes take place.
* **Customer Table** - An AWS Dynamo DB table that stores the the Customer Account data mentioned above.

## App Flows

We have the flexibility to utilize different app platforms such as iOS, Android, and web-based applications for our examples. AWS provides extensive SDKs that cover a wide range of platforms, including iOS, Android, web, and more.

* [AWS SDK for iOS](https://github.com/aws-amplify/aws-sdk-ios)
* [AWS SDK for Android](https://github.com/aws-amplify/aws-sdk-android)
* [AWS SDK for JavaScript](https://aws.amazon.com/sdk-for-javascript/) 

 We will assume that ACME utilizes AWS Cognito as its identity provider. Customers are registered and their identity records are securely stored, with a cross-reference to the customer account ID data mentioned earlier, ensuring seamless integration and data consistency.

### Subscribing to Updates

To minimize the notification overload for the app regarding customer account edits, we will take advantage of AWS [SNS message filtering](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html). This powerful feature allows us to filter out irrelevant events. By implementing a Universally Unique Identifier (UUID) as the customer ID and enforcing authentication for users, we significantly reduce the likelihood of account event cross-contamination.

It is important to highlight that the flow for both the representative app and the customer app is identical. In the sequence diagrams presented below, the term "App" encompasses both the representative and customer applications.

```mermaid
sequenceDiagram
  title App Subscription Flow
  App->>SNS SDK: Subscribe to events filtered on the customer id
  SNS SDK->>Live Topic: Subscribe to live topic
  Live Topic->>SNS SDK: Return status
  SNS SDK->>App: Return status
```

```mermaid
sequenceDiagram
  title App Receives Live Edit Event Flow
  Live topic->>SNS SDK: Event published for filtered customer id
  SNS SDK->>App: Changed data attributes received
  App->>App: View / Page Updates to Reflect Changes

```

### Published Live Edit Events

To publish event data on the Live topic, we'll utilize [Dynanmo DB streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/streamsmain.html). This feature operates akin to data triggers. By configuring the stream, we can publish changes to account data based on the customer ID key.

```mermaid
sequenceDiagram
  title Publish Account Changes to Live topic
  App->>ACME Server API: Updated customer data
  ACME Server API->>Customer Table: Data persisted in table
  Customer Table->>Table Stream: Save triggers event stream
  Table Stream->>Live topic: Updated data put on topic
```

## Thoughts and Comments

The architecture presented above offers a method for real-time data editing. We encourage the submission of any additional ideas on this topic.
