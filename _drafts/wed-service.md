---
layout: post
title: Web Service Introduction
categories: Java EE
description: introduction for web service applications
keywords: web, service
---

## What is web service?
* Designed for machine-to-machine (or allication-to-application) interaction
* Should be interoperatble - Not platform dependent
* Should allow communication over a network

## How?
* Data exchange:
   * Application send `request` to Web Service, and Web Servic send `response` back to the Application. 
* Make web services platform independent:
   * Make `request` and `response` platform independent. Use the following two formats:
     * XML
     * JSON
* Application know the format of `request` and `response`:
  * Service Definition
    * request/response format
    * request structure
    * response structure
    * endpoint

## Terminology
1. Request: The input of the service.
2. Response: The output of the service.
3. Message Exchage Format: The format of the request and the response. (XML / JSON)
4. Service provider: The one which hosts the web service, also named Server.
5. Service consumer: The one which is consuming the web service, also named Client.
6. Service definition: The contract between server and client.
7. Transport: Defines how a service is called. (HTTP / MQ)

## Web service groups
* SOAP - Simple Object Access Protocol
  * Format:
    * SOAP XML Request
    * SOAP XML Response
  * Transport:
    * SOAP over MQ
    * SOAP over HTTP
  * Service Definition
    * WSDL (web service definition language)
      * Endpoint
      * All Operations
      * Request Structure
      * Response Structure
* REST - represenational state transfer
  * key abstraction: resource
    * resource is anything that you want to expose to the outside world through your application.
    * A resource has an URI (Uniform Resource Identifier).
    * A resource can have different representations.
      * XML
      * HTML
      * JSON
    * Data exchange foramt:
      * No restriction, JSON is popular.
    * Transport
      * Only HTTP
    * Service Definition
      * No standard. WADL/Swagger/...

## REST vs SOAP
1. SOAP is a format of XML, while REST is actually an architectural style.
2. In SOAP, the data exchange is always XML(the SOAP XML with SOAP envelope header and body). In REST, there is no strict data exchange format.
3. As for the service definition, SOAP uses WSDL, REST doesn't have a standard definition language.
4. As for the transport, SOAP has no restriction (HTTP / MQ). REST is very specific about making the best use of HTTP protocol.
5. RESTful services is easier to implement than SOAP.

## Reference
Udemy, [Master Java Web Services and RESTful API with Spring Boot](https://www.udemy.com/share/101Yoi3@3Hybns8LYEYORtVnGN1EJGttb1pQ6nA73IsFL6R8e8GXxPqurSv4AX9KcShy17NSAw==/)
