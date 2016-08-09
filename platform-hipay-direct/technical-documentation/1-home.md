#Payment Gateway - API Integration Guide

##Document Conventions
These conventions help to locate and interpreted information.
This guide uses several typographic conventions to highlight certain words, phrases and
point out specific pieces of information.
The following table clarifies the conventions used across this guide.

![legende](images/001.png)

##Abbreviations used in tables
The tables of this document describe data elements. These data elements are equivalent
to parameters in a query or to fields of a response message. The following table helps
understanding the different attributes (columns) that define a data element.

![legende](images/002.png)

##Acronyms and Abbreviations
The following acronyms and abbreviations are used in this guide.

![legende](images/003.png)

#Chapter 1. API Overview

The API is based on SOAP principles; thus it is very easy to write and test applications.
How to access the API:

- Use your browser to access URLs,.
- Use almost any HTTP client in any programming language, to interact with the API.

##1.1 Security Considerations
HiPay Direct SOAP API service is protected to:

- ensure that only authorized Merchants use it,
- prevent payment information from being compromised

###Encrypted Communication

![legende](images/004.png)

#Chapter 2. Integration Guidelines
This chapter outlines the basic integration requirements that you app must meet.

##2.1 Submitting a Request

###Service Endpoints
There are two endpoints (base URLs) that you can make your API calls to.

- Stage if you are testing your integration, and
- Production for when you have finished testing and want your application to go live.
All URLs referenced in this guide must have one of the following bases:


**Stage**: https://test-ws.hipay.com/

**Production**: https://ws.hipay.com/

###Authentication

![legende](images/005.png)

###Character Encoding
All parameters accept UTF-8 encoded text via the API. All other encodings must be
converted to UTF-8 before sending them to the HiPay API in order to guarantee that the
data is not corrupted.

##2.2 Response Handling
Whether a request succeeded or not, it is indicated by the HTTP status code. A 2xx status
code indicates a success, whereas a 5xx status code indicates a failure.

###Response Status Codes
The HiPay API attempts to return appropriate HTTP status codes for every request.
These are the HTTP status codes you may receive and their meaning.

![legende](images/006.png)

###Response Format

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAPENV="http://schemas.xmlsoap.org/soap/envelope/"
xmlns:ns1="https://ws.hipay.com/soap/payment-v2">
<SOAP-ENV:Body>
<ns1:generateResponse>
<generateResult>
<redirectUrl>https://payment.hipay.com/index/pay/payid/716…7106fc</redirectUrl>
<code>0</code>
<description>N/A</description>
</generateResult>
</ns1:generateResponse>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##2.3 Error Handling

![legende](images/007.png)