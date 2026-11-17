## 1.Introduction

Web application security is an important part of cybersecurity because web applications may contain different security weaknesses that could affect their confidentiality, integrity, and availability.

This project focuses on performing a security assessment of the Damn Vulnerable Web Application (DVWA), which is an intentionally vulnerable web application designed for cybersecurity training and practical testing.

The main purpose of this phase is to understand the application’s structure, identify its available pages and endpoints, and analyze the HTTP requests exchanged between the browser and the web application.

The assessment is performed inside a controlled local environment using Kali Linux, Docker, and Burp Suite.

## 2.Project Objectives

## The main objectives of this phase are:

- 1. Prepare a working DVWA testing environment.

- 2. Configure the required security testing tools.

- 3. Explore the application’s structure and available functionalities.

- 4. Identify the application’s pages and endpoints.

- 5. Capture and analyze HTTP requests.

- 6. Identify HTTP methods such as GET and POST.

- 7. Document request parameters.

- 8. Create a Sitemap and Application Map.

- 9. Prepare organized documentation for the next testing phases.

## 3.Testing Environment Setup

In this phase, the testing environment was prepared to provide a safe and controlled environment for the assessment.

The environment consists of the following components:

## Component

- Kali Linux

- Docker

- DVWA

- Burp Suite

## Description

Operating system used for security testing.

Used to run the DVWA application.

Target web application.

Used to intercept and analyze HTTP traffic.


- Browser

- draw.io

## 3.1 DVWA Installation

DVWA was deployed locally using Docker to create an isolated testing environment.

After starting the Docker container, the application was accessed through the local browser to verify that it was running correctly.

## 4.Tools Configuration

Several tools were prepared and configured to support the application mapping process.

## 4.1 Kali Linux

Kali Linux was used as the main operating system for the testing activities.

## 4.2 Burp Suite

Burp Suite was configured as an intercepting proxy to capture and inspect the HTTP traffic exchanged between the browser and DVWA.

## It helped identify:

- HTTP request methods.

- Request URLs.

- Request parameters.

- HTTP headers.

- Server responses.

## 4.3 Browser Configuration


The browser was configured to communicate through Burp Suite, allowing the captured application traffic to be reviewed.

## 5.Application Discovery

After preparing the testing environment, the application was explored through its available pages and functionalities.

The purpose of this activity was to understand how the application is organized and identify the different sections accessible through its interface.

During this process, the available application pages and their corresponding requests were observed and documented.

## 5.1 Application Sitemap

The Burp Suite Sitemap was used to organize the discovered application paths.

It provides an overview of the available resources and helps understand the application’s structure.

## 6.Application Mapping


After collecting the required application information, an Application Map was created using draw.io.

The map represents the application’s main structure and the relationship between its different sections.

## It includes:

- Main Application.

- Login Page.

- Home Page.

- Security Settings.

- Instructions.

- Vulnerability Modules.

- Identified Endpoints.

The purpose of this map is to provide a clear visual representation of the application and make the next testing activities easier to organize.

## 7.HTTP Request Analysis

During the application discovery process, HTTP requests were captured and reviewed using Burp Suite.

Each request was analyzed to identify its main components.

The analysis focused on:

## 7.1 HTTP Methods

HTTP methods specify the type of operation requested by the client.

## The main methods observed during this phase include:

- GET: Used to request resources from the server.

- POST: Used to submit data to the server.

## 7.2 Request Parameters

Parameters represent the data sent with an HTTP request.

They were reviewed to understand what information is submitted to each endpoint.


## 7.3 Endpoint Identification

Each discovered endpoint was recorded to build an organized inventory of the application’s accessible resources.

## 8.Endpoint Inventory

An Excel spreadsheet was created to organize the discovered endpoints and their corresponding HTTP requests.

## The spreadsheet contains information such as:

## Field Field

- Endpoint

- HTTP Method HTTP Method

- Parameters

- Parameter Count

- Notes

## 9.Evidence Collection

Evidence collection is an important part of the assessment because it provides visual proof of the activities performed during the project.

## Description Description

Application request path.

GET or POST.

GET or POS

Parameters included in the request.

Total number of identified parameters.

Additional observations.

## The collected evidence includes:

- 1. DVWA running locally.

- 2. Docker container status.

- 3. Burp Suite configuration.

- 4. Application Sitemap.

- 5. Application Map.

- 6. Captured HTTP requests.

- 7. Endpoint Inventory.


## 10.Results and Observations

During the application mapping phase, several important activities were completed.

The results include:

- Successful preparation of the local DVWA environment.

- Successful configuration of Burp Suite.

- Identification of the application’s main sections.

- Collection of application paths through the Sitemap.

- Identification of HTTP request methods.

- Initial analysis of request parameters.

- Creation of an organized Endpoint Inventory.

- Development of a visual Application Map.

These results provide a foundation for the next phases of the security assessment.

## 11.Conclusion

The Application Mapping phase provided a better understanding of the DVWA application’s structure and communication with the server.

By using Burp Suite, the application’s HTTP traffic was captured and analyzed, while the Sitemap and Application Map helped organize the discovered resources and navigation paths.

The collected information was documented in an Excel spreadsheet to provide a structured reference for the identified endpoints, HTTP methods, and request parameters.

Completing this phase establishes the necessary foundation for continuing the security assessment and performing further security testing within the authorized laboratory environment.
