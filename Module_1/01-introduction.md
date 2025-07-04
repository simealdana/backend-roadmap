# Introduction to REST APIs

In this first intensive session, we will address what a REST API is and its fundamental concepts. Given that our student has a background in mechatronics engineering and basic knowledge of C++, we will focus the class on the architecture and principles of REST APIs rather than code writing. At the end, we will propose a practical challenge to consolidate what we've learned. Let's start with the basics.

## What is an API?

An API (Application Programming Interface) is an interface that defines how different software applications interact with each other. In simple terms, an API is a set of definitions and rules that acts as a contract between a client (who requests data or functions) and a provider (who offers that data or functions). 

For example, a weather service API might require the client to send a postal code, and in return, the service responds with data such as maximum and minimum temperature for that area. APIs then allow data exchange or function execution in a structured way: the client doesn't need to know internal details of how the server obtains the information, they just need to follow the established contract (request and response format). 

In summary, an API is the mediator between users or clients and the resources/services of a system, allowing secure and controlled application integration. Many popular applications expose APIs so that third-party developers can interact with their data or functionalities (for example, Twitter, Google Maps, Facebook, etc.).

## What is REST?

REST stands for **Representational State Transfer**. It's not a protocol or a concrete standard, but rather a software architecture style defined by Roy Fielding in 2000 as part of his doctoral thesis. 

REST consists of a set of design principles and best practices for distributed systems (particularly for web services) that leverage the same rules that have made the Web successful. In other words, an API that follows REST principles is called "RESTful" and uses Web conventions (HTTP, URLs, etc.) consistently.

> **Note**: In practice, the term REST API is often used broadly to refer to any web API that uses HTTP to transport data (usually JSON) without an additional layer like SOAP. However, strictly speaking, a truly RESTful API should adhere to certain specific architectural principles that we will detail in the next section.

## Key Takeaways

- **API**: A contract between client and server for data/function exchange
- **REST**: An architectural style, not a protocol
- **RESTful**: An API that follows REST principles
- **HTTP**: The underlying protocol that REST APIs use
- **JSON**: The most common data format for modern REST APIs

## Next Steps

In the following sections, we'll explore:
- The 6 core principles of REST architecture
- How REST APIs work in practice
- HTTP status codes and their meanings
- Tools for working with REST APIs 