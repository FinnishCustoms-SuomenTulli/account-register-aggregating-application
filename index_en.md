[Tiedon hyödyntäjien käyttöönoton ja ylläpidon ohje, koostava sovellus](instructions/Tiedon_hyödyntäjien_käyttöönoton_ja_ylläpidon_ohje_koostava_sovellus.pdf)  
[Koostavan sovelluksen rajapintakuvaus](index.md)  
[Deployment and maintenance instructions for data users, Aggregating Application](instructions/Deployment_and_maintenance_instructions_for_data_users_Aggregating_Application.pdf)  
[Instruktioner för uppgiftsanvändare om ibruktagande och underhåll, sammanställningsprogrammet](instructions/Instruktioner_för_uppgiftsanvändare_om_ibruktagande_och_underhåll_sammanställningsprogrammet.pdf)  
[Beskrivning av sammanställningsprogrammets frågegränssnitt](index_sv.md)  

# Description of the aggregating application’s query API

*Document version 1.02*

## Version history

| Version | Date       | Description                                                        |
|---------|------------|--------------------------------------------------------------------|
| 1.0     | 7.2.2023 | Version 1.0                                                        |
| 1.01    | 29.6.2023 | Added schema for fin.012 and a new example message. Updated chapter 4.2 with two new data elements. One is used to aim the query to specific datasource(s), the other to mark that the query is related to an international/cross-border information request. Also updated chapter 4.7 with use of InvstgtnSts NOAP in response messages. |
| 1.02    | 26.4.2024 | Clarification to chapter 2 that status API returns COMP also when no hits were found. |

## Table of contents

1. [Introduction](#chapter1)  
2. [Query for bank and payment account information from the aggregating application](#chapter2)  
3. [Information security](#chapter3)  
4. [Query API of the aggregating application](#chapter4)   
  4.1 [Message structure of the SOAP operations of the query API](#4-1)    
  4.2 [Query API](#4-2)    
  4.3 [Status API](#4-3)    
  4.4 [Result API](#4-4)    
  4.5 [Message extension Fin020 (QueryResultRequest)](#4-5)    
  4.6 [Message extension Fin021 (QueryResultResponse)](#4-6)    
  4.7 [Use of the ReturnIndicator1 element](#4-7)    
  4.8 [Error management](#4-8)    
  4.9 [Example messages](#4-9)    
  

## 1. Introduction <a name="chapter1"></a>

### 1.1 Terms and abbreviations

| Abbreviation or term | Definition                                                                                                                                                                                                  |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Interface            | A standard practice or connection point that allows the transfer of information between devices, programmes and the user.                                                                                   |
| WS (Web Service)     | Software operating in a network server, providing services for use by applications through standardised internet connection practices. The data retrieval system provides information queries as a service. |
| Endpoint             | An interface service available at a certain network address.                                                                                                                                                |
| WSDL                 | (Web Service Description Language) A structural description language describing the functionalities provided by the web service.                                                                            |

### 1.2 Purpose and scope of the document

This document supplements the regulation issued by Finnish Customs on the Bank and Payment Accounts Control System. The purpose of this document is to issue instructions for the query API of the aggregating application. This document is supplemented by the deployment and maintenance instructions for data users.

### 1.3 References

[Data retrieval system WSDL](https://finnishcustoms-suomentulli.github.io/account-register-information-query/wsdl/data-retrieval-system-wsdl.xml)

[fin.020.001.01](schemas/fin.020.001.01.xsd)

[fin.021.001.03](schemas/fin.021.001.03.xsd)

[fin.012.001.03](schemas/fin.012.001.03.xsd)

[Guidelines on the information security of e-services](http://julkaisut.valtioneuvosto.fi/bitstream/handle/10024/80012/VM_25_2017.pdf)

### 1.4 General description

Finnish Customs has established an Account Register Project to implement the bank and payment account monitoring system, based on Finnish legislation and implementing EU Directive 2018/843.

This document describes the aggregating application’s APIs.

## 2. Query for bank and payment account information from the aggregating application <a name="chapter2"></a>

This chapter describes the query of bank and payment account information from the aggregating application.

Figure 2.1 presents the query process as a flow diagram.

![Query for information from the aggregating application](diagrams/flowchart_koostava.png "Query for information from the aggregating application")  
*__Figure 2.1.__ Query for information from the aggregating application*  

Table 2.2 presents the meaning of different variables in the flow diagram. 

*__Table 2.1.__ Variables in the flow diagram*

| Muuttuja           | Kuvaus                                                                                                                                                                                                                                                                                                    |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| POLLING_INTERVAL   | Polling interval; the time the client has to wait before the next query is 1 minute. If a client’s polling interval is too short, the server may reject transaction processing (error code 3, see [Table 4.12.1](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#4-12)). |
| POLLING_TIME_LIMIT | The permitted time limit for polling, after which it will be stopped. If no response is still received, a new query must be made or the case must be transferred to manual processing.                                                                                                                    |

The flow of the query is as follows:
1. The client sends a query message to the query API.
2. The query API returns a key as a response (resultKey).
3. The client waits for a while (see POLLING_INTERVAL) and sends a status query containing the key to the status API.
4. The status API either  
  a. returns code NRES if the results are not yet ready, or  
  b. returns code COMP if the results are ready. If the query produced any hits, a list of keys is also returned. 
5. If the code is  
  a. NRES, the client returns to step 3.  
  b. COMP and hits were found, the client sends a search result query to the result API using one of the keys received in step 4 b.
  c. COMP but no hits, the process will end.
7. The result API returns a search result message corresponding to the key that was sent.
8. If there still are search results waiting for retrieval, the process will return to step 6 to repeat the search using the next key.
9. If all search results have been received, the process will end.
 
The codes to be returned are defined in ISO code set StatusResponse1Code, and the use of its values is described in Table 2.2.

*__Table 2.2.__ Use of StatusResponse1Code values*

| Code | Name             | Definition                      | Description                                                                      |
|:---|:---|:---|:---|
| COMP | CompleteResponse | Response is complete.           | The query has been completed and possible results are available.                 |
| NRES | NoResponseYet    | Response not provided yet.      | The response message does not include retrieval results; make a new query later. |
| PART | PartialResponse  | Response is partially provided. | Not used.                                                                        |


#### Result retention time in the aggregating application
Results are deleted once they have been retrieved. Complete results are retained for at most 24 hours after their completion, during which the results must be retrieved. 

## <a name="chapter3"></a> 3. Information security

Information security practices follow the same principles described in the specification of the [query API of the data retrieval system](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#information_security).

## <a name="chapter4"></a> 4. Aggregating application API

The API will be implemented as a SOAP/XML web service, whose [WSDL](https://finnishcustoms-suomentulli.github.io/account-register-information-query/wsdl/data-retrieval-system-wsdl.xml) has been published in conjunction with the query API description of the data retrieval system.

SOAP protocol version 1.1 will be used.

ISO 20022 code set references will be used in the messages. The code sets are available on the ISO 20022 page [External Code Sets](https://finnishcustoms-suomentulli.github.io/account-register-information-query/assets/iso20022org/ExternalCodeSets_2Q2020_August2020_v1.xlsx).

The aggregating application API consists of three endpoints: 
- [for receiving queries](#4-2)
- [for requesting the query status](#4-3)
- [for retrieving search results](#4-4)

### <a name="4-1"></a> 4.1 Message structure of the SOAP operations of the query API
The message structure used in the API is identical at the main level to the specification of the [query API of the data retrieval system](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#queryinterface) published by Finnish Customs.
In addition, the [fin.020](#4-5) and [fin.021](#4-6) submessages have been defined for the API to transmit the identifiers required to check the retrieval status and retrieve results.

### <a name="4-2"></a> 4.2 Query API
Messages sent to the query API are similar to the messages used in the [query API of the data retrieval system](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#kyselyrajapinta). Message extension fin012 has two additional elements that are not used in data retrieval systems' messages: RequestedDataSources and InternationalRequest. RequestedDataSources element is used to aim the query to one or several data sources (data retrieval systems and account register). The query with RequestedDataSources element is only forwarded to data source(s) specified in the element. InternationalRequest element is used to mark that the query is related to an international/cross-border information request based on the directive on using financial information ((EU) 2019/1153) article 19 and national anti-money laundering act (444/2017) 4 § in chapter 2.

Response messages also follow the same structure, while the fin.021 message is used as a submessage in the auth.002 message to indicate the identifier of the received query. This identifier is used to check the query status in the [status API](#4-3). In status API, other submessages return status NFOU, which is insignificant in this case.

#### <a name="fin012"></a> 4.2.1 Message extension InformationRequestFIN012

The submessage schema is defined in the [fin.012](schemas/fin.012.001.03.xsd) file.
The message extension is appended to the Xpath location of the ISO 20022 message listed in the table.

| Name                                             | [min..max] | Type                                           | Description                                                                                                                             | Liitetään sanomaan | XPath                                    |
|:-------------------------------------------------|:-----------|:-----------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------|:-------------------|:-----------------------------------------|
| InformationRequestFIN012                         |            |                                                |                                                                                                                                         | auth.001           | `/Document/InfReqOpng/SplmtryData/Envlp` |
| &nbsp;&nbsp;&nbsp;&nbsp;AuthorityInquiry         | [1..1]     | [AuthorityInquirySet](#authority-inquiry-set)  | Authority details associated with the query                                                                                             |                    |
| &nbsp;&nbsp;&nbsp;&nbsp;AdditionalSearchCriteria | [0..\*]    |                                                | Used for the search by safety-deposit box ID.                                                                                           |                    |
| &nbsp;&nbsp;&nbsp;&nbsp;RequestedDataSources     | [0..1]     | [RequestedDataSources](#requested-datasources) | Datasource or datasources that will receive the query. If the element is missing from the message, the query is sent to all datasources. |                    |
| &nbsp;&nbsp;&nbsp;&nbsp;InternationalRequest     | [0..1]     | boolean                                        | Value "true" is used if the query is related to an international information request.                                                    |                    |

#### <a name="authority-inquiry-set"></a> AuthorityInquirySet

| Name                                       | Type       | In use | Description                                         |
|:-------------------------------------------|:-----------|:-------|:----------------------------------------------------|
| AuthorityInquirySet                        |            |        |                                                     |
| &nbsp;&nbsp;&nbsp;&nbsp;OfficialId         | Max140Text | Yes    | Identifier of the official that is making the query |
| &nbsp;&nbsp;&nbsp;&nbsp;OfficialSuperiorId | Max140Text | Yes    | Identifier of the official's superior               |

#### <a name="requested-datasources"></a> RequestedDataSources

| Name                                    | [min..max] | Type      | Description                         |
|:----------------------------------------|:-----------|:----------|:------------------------------------|
| RequestedDataSources                    |            |           |                                     |
| &nbsp;&nbsp;&nbsp;&nbsp;DataSourceOrgId | [1..\*]    | Max35Text | Datasource's business identity code |

### <a name="4-3"></a> 4.3 Status API
Messages sent to the status API consist of the same message content as queries. In addition, the identifier received as the query API’s response is transmitted through the [fin.020 submessage](#4-5).

Response messages are identical to the [query API's](#4-2) responses, plus: 
- If a query is not yet complete, NRES will be returned as the status code in the Auth.002 schema’s RspnSts field.
- If a query is complete, COMP will be returned as the status code in the Auth.002 schema’s RspnSts field along with the identifier used in the status query in the [fin.021 submessage](#4-6) and a list of identifiers using which results can be retrieved from the [result API](#4-4). 

### <a name="4-4"></a> 4.4 Result API
Messages sent to the result API consist of the same message content as queries. In addition, the identifiers received as the status API’s response are transmitted one-by-one through the [fin.020 submessage](#4-5).

Response messages are identical to the [response messages](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#InformationRequestResponseV01) of the data retrieval system’s query API, containing actual search results in accordance with the schema. In addition, the result identifier is returned in the [fin.021 submessage](#4-6) and the business ID of the data retrieval system that returned the result is returned in the _datasourceIdentifier_ attribute.

### <a name="4-5"></a> 4.5 Message extension Fin020 (QueryResultRequest)
The submessage schema is defined in the [fin.020](schemas/fin.020.001.01.xsd) file. The message extension is appended to the Xpath location of the ISO 20022 message listed in the table.

|Name|[min..max]|Type|Description|Appended to message|XPath|
|:---|:---|:---|:---|:---|:---|
|QueryResultRequest| | | |[auth.001](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#InformationRequestOpeningV01)|`/Document/InfReqOpng/SplmtryData/Envlp`|
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKeyList|[1..1]|ResultKeyList|A list of the identifiers of returned information||

|Name|[min..max]|Type|Description|Notes|
|:---|:---|:---|:---|:---|
|ResultKeyList| | | ||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKey|[1..n]|Max256Text|UUID of the query or result|Only one value at a time is supported for now|

### <a name="4-6"></a> 4.6 Message extension Fin021 (QueryResultResponse)
The submessage schema is defined in the [fin.021](schemas/fin.021.001.03.xsd) file. 
The submessage is returned in the [data retrieval system’s response message](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#InformationRequestResponseV01) in the [ReturnIndicator1](#4-7) element similarly to other submessages.

The following data is returned as attributes of the ResultKeyList element:
- datasourceOrganisationId: Business id or VAT id of the datasource 
- errorCode: in case of error, either the error code returned from the datasource or one generated by Aggregating application, see [4.8 Error management](#4-8)  

|Name|[min..max]|Type|Description|Appended to message|XPath|
|:---|:---|:---|:---|:---|:---|
|QueryResultResponse| | | |[auth.002](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#InformationRequestResponseV01)|`/Document/InfReqRspn/RtrInd/InvstgtnRslt/Rslt`|
|&nbsp;&nbsp;&nbsp;&nbsp;QueryKeyList|[0..1]|QueryKeyList|A list of the identifiers used in query||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKeyList|[0..1]|ResultKeyList|A list of the identifiers of returned information||


|Name|[min..max]|Type| Description                                          |Notes|
|:---|:---|:---|:-----------------------------------------------------|:---|
|QueryKeyList| | |                                                      ||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKey|[1..n]|Max256Text| UUID of the result                                   ||
|ResultKeyList| | |                                                      ||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKey|[1..n]|Max256TextAllowedEmpty| UUID of the result or empty in case of a query error |Optional attributes: `datasourceOrganisationId` and `errorCode`|

### <a name="4-7"></a> 4.7 Use of the ReturnIndicator1 element

ReturnIndicator1 includes the presence of a single type of search result as in the [response messages](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#InformationRequestResponseV01) of the data retrieval system’s query API. The fin.021 submessage is returned in this element similarly to other submessages returned in the query.

| XPath                       | Type                       | Description                                                                                                                                                                                                                                  |
|:----------------------------|:---------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| RtrInd/AuthrtyReqTp/MsgNmId | Max35Text                  | Includes the message ID of the message extension (fin.021.001.03)) |
| RtrInd/InvstgtnRslt         | InvestigationResult1Choice | Returning `Rslt` element type SupplementaryDataEnvelope1, which includes one of the submessages ([fin.021](#4-6), supl.027, fin.013 or fin.002) or the `InvstgtnSts` element. `InvstgtnSts` is returned using code `NFOU`, if no information can be found using the identifier used in the query. `InvstgtnSts` is returned using code `NOAP` in case a query with the name of a natural or legal person has multiple hits in one party's data in account register. |

### <a name="4-8"></a> 4.8 Error management
Error management and returned codes follow the specifications of the [WS message traffic scenarios in the query API](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#4-12) chapter for the data retrieval system’s query API, where applicable.

The response from Status API can be an empty ResultKey along with an errorCode to indicate that something has gone wrong with the query. No datasources have been queried at this point and there won't be any results to fetch. Currently "TIMEOUT" is the only produced errorCode in this case.

The responses from Status and Result APIs include also information of possible errors concerning certain datasources. This information is returned in `errorCode` attribute of the ResultKey element.

It is not possible to query information concerning time period before September 1st 2020 from the aggregating application. The query API rejects queries where investigation period (InvstgtnPrd) is before that.

### <a name="4-9"></a> 4.9 Example messages
Examples of each query message and their responses are available in the examples folder:
- [Query message](examples/query1.xml) and [response](examples/query1_response.xml)
- [Status query](examples/status1.xml), [response](examples/status1_response.xml) and [Query error response](examples/status1_response_with_query_error.xml).
- [Result query](examples/result1.xml), [response](examples/result1_response.xml) and [response with aimed query and international information request](examples/query_with_requested_datasources_and_international_request.xml).
  
