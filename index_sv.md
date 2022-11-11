[Tiedonhakujärjestelmän kyselyrajapintakuvaus](https://finnishcustoms-suomentulli.github.io/account-register-information-query/)  
[Query interface description of the data retrieval system](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html)  
[Beskrivning av datasöksystemets frågegränssnitt](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_sv.html)

# Beskrivning av sammanställningsprogrammets frågegränssnitt

*Dokumentversion 1.0*

## Versionhistorik

Version|Datum|Beskrivning
---|---|---
1.0|24.10.2022|Version 1.0


## Innehåll

1. [Inledning](#chapter1)  
2. [Förfrågan om bank- och betalkontouppgifter i sammanställningsprogrammet](#chapter2)  
3. [Datasäkerhet](#chapter3)  
4. [Sammanställningsprogrammets frågegränssnitt](#chapter4)   
  4.1 [Meddelandestruktur för frågegränssnittets SOAP-operationer](#4-1)    
  4.2 [Frågegränssnitt](#4-2)    
  4.3 [Statusgränssnitt](#4-3)    
  4.4 [Resultatgränssnitt](#4-4)    
  4.5 [Utvidgat meddelande Fin020 (QueryResultRequest)](#4-5)    
  4.6 [Utvidgat meddelande Fin021 (QueryResultResponse)](#4-6)    
  4.7 [Användning av elementet ReturnIndicator1 med fin.021-meddelande](#4-7)    
  4.8 [Hantering av felsituationer](#4-8)    
  4.9 [Exempelmeddelanden](#4-9)    
  

## 1. Inledning <a name="chapter1"></a>

### 1.1 Termer och förkortningar

Förkortning eller term|Förklaring
---|---
Gränssnitt|Standardenlig praxis eller kontaktyta som möjliggör överföring av information mellan enheter, programvaror eller användare. 
WS (Web Service)|Webbaserat datorprogram som med hjälp av standardiserade internetprotokoll ställer tjänster till förfogande för applikationerna. Tjänsten som datasöksystemet tillhandahåller är förfrågan om uppgifter.
Endpoint|Gränssnittstjänst som finns tillgänglig på en viss webbadress.
WSDL|(Web Service Description Language) Strukturellt beskrivningsspråk som används för att beskriva funktioner som en webbtjänst erbjuder.

### 1.2 Dokumentets syfte och omfattning

Detta dokument kompletterar Tullens föreskrift om ett övervakningssystem för bank- och betalkonton. Syftet med dokumentet är att ge anvisningar om sammanställningsprogrammets frågegränssnitt. Detta dokument kompletteras av Instruktioner för ibruktagande och underhåll för uppgiftsanvändare.

### 1.3 Hänvisningar

[WSDL för datasöksystemet](https://finnishcustoms-suomentulli.github.io/account-register-information-query/wsdl/data-retrieval-system-wsdl.xml)

[fin.020.001.01](schemas/fin.020.001.01.xsd)

[fin.021.001.01](schemas/fin.021.001.01.xsd)

[Anvisning om informationssäkerheten inom elektronisk ärendehantering](http://julkaisut.valtioneuvosto.fi/bitstream/handle/10024/80012/VM_25_2017.pdf)

### 1.4 Allmän beskrivning

Tullen har inrättat ett Kontoregisterprojekt för att genomföra direktivet (EU) 2018/843 och det övervakningssystem över bank- och betalkonton som grundar sig på Finlands lagstiftning och vars syfte är att genomföra direktivet.

I detta dokument beskrivs sammanställningsprogrammets frågegränssnitt.

## 2. Förfrågan om bank- och betalkontouppgifter i sammanställningsprogrammet <a name="chapter2"></a>

I detta kapitel beskrivs förfrågan om bank- och betalkontouppgifter i sammanställningsprogrammet.

Frågeprocessen åskådliggörs som ett flödesschema i bild 2.1.

![Uppgiftsförfrågan i sammanställningsprogrammet](diagrams/flowchart_koostava.png "Uppgiftsförfrågan i sammanställningsprogrammet")  
*__Bild 2.1.__ Uppgiftsförfrågan i sammanställningsprogrammet*  

I tabell 2.1 redogörs för parametrarna i flödesschemat. 

*__Tabell 2.1.__ Parametrarna i flödesschemat*

Parameter|Beskrivning
---|---
POLLING_INTERVAL|Pollingintervall, ett tidsintervall som klienten måste vänta ut före följande förfrågning. Om klienten pollar servern för frekvent, kan servern avvisa behandlingen av transaktionen (felkod 3, se [Tabell 4.12.1](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#4-12)).
POLLING_TIME_LIMIT|Maximal tid för pollingar, innan man slutar. Om inget svar fås, ska man göra en helt ny förfrågan eller överföra ärendet till manuell handläggning.

Värdena för gällande parameter som anges i tabell 2.1 anges i den bifogade dokumentationen.

Förfrågningen har följande flöde:
1. Klienten skickar ett frågemeddelande till Query API.
2. Som svar returnerar Query API en nyckel (resultKey).
3. Klienten väntar en stund (se POLLING_INTERVAL) och skickar sedan en statusförfrågan med nyckeln till Status API.
4. Status API returnerar  
  a. antingen koden NRES, om svaret inte ännu är färdigt  
  b. eller koden COMP och en lista på nyckel/datakälla-par. 
5. Om koden är NRES, går klienten tillbaka till punkt 3.
6. Om koden är COMP, skickar klienten en förfrågan på sökresultat till Result API med en av de nycklar som den mottagit i punkt 4.b.
7. Result API returnerar ett meddelande med det sökresultat som motsvarar nyckeln.
8. Om alla sökresultat inte har hämtats, återgår man till punkt 6 och upprepar sökningen med nästa nyckel.
9. Om alla sökresultat har hämtats, gå till slut.
 
Responskoderna definieras i ISO-koduppsättningen StatusResponse1Code; användningen av dem beskrivs i tabell 2.2.

*__Tabell 2.2.__ Användning av värden för StatusResponse1Code*

|Kod|Namn|Definition|Beskrivning|
|:--|:--|:--|:--|
|COMP|CompleteResponse|Response is complete.|Svarsmeddelandet innehåller sökresultaten.|
|NRES|NoResponseYet|Response not provided yet.|Svarsmeddelandet innehåller inga sökresultat, gör en ny förfrågan senare.|
|PART|PartialResponse|Response is partially provided.|Används inte.|

#### Förvaringstid för resultaten i sammanställningsprogrammet
De färdiga resultaten förvaras högst en specificerad tid efter att de sammanställts och de ska hämtas under denna tid. Resultaten raderas inte när de hämtas, utan först när nämnda tidsfrist har löpt ut. Värdet för tidsfristen anges i den bifogade dokumentationen.

## <a name="chapter3"></a> 3. Datasäkerhet

Datasäkerhetsförfarandet följer de principer som beskrivs i [beskrivning av datasöksystemets frågegränssnitt](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_sv.html#dataskydd).

## <a name="chapter4"></a> 4. Sammanställningsprogrammets frågegränssnitt

Gränssnittet genomförs med SOAP/XML Web Service, för vilken [WSDL](https://finnishcustoms-suomentulli.github.io/account-register-information-query/wsdl/data-retrieval-system-wsdl.xml) publicerats med beskrivning av datasöksystemets frågegränssnitt.

Version 1.1 av SOAP-protokollet används.

I meddelandena används hänvisningar till koduppsättningen i ISO 20022-standarden. Hänvisningarna till koduppsättningen finns på ISO 20022 sidan [External Code Sets](https://finnishcustoms-suomentulli.github.io/account-register-information-query/assets/iso20022org/ExternalCodeSets_2Q2020_August2020_v1.xlsx).

Sammanställningsprogrammets gränssnitt har följande tre endpoint: 
- [mottagning av förfrågan](#4-2)
- [förfrågan om förfrågans status](#4-3)
- [hämtning av sökresultat](#4-4)

### <a name="4-1"></a> 4.1 Meddelandestruktur för frågegränssnittets SOAP-operationer
Den meddelandestruktur som används i gränssnittet är på huvudnivån identisk med definitionen i Tullens [frågegränssnitt för datasöksystemen](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_sv.html#fragegranssnitt).
I gränssnittet har dessutom definierats undermeddelandena [fin.020](#4-5) och [fin.021](#4-6), för förmedling av de nödvändiga koderna för kontroll av sökningens status och hämtning av sökresultaten.

### <a name="4-2"></a> 4.2 Frågegränssnitt
Meddelandet som skickas till frågegränssnittet är identiskt med meddelandet i [frågegränssnittet för datasöksystemen](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_sv.html#fragegranssnitt).

Svarsmeddelandet har också samma struktur, men meddelandet auth.002 innehåller undermeddelandet fin.021, som anger koden för den förfrågan som mottagits. Koden används vid kontroll av förfrågans status i [statusgränssnittet](#4-3). 

De övriga undermeddelandena returnerar status NFOU, som i detta fall saknar betydelse.

### <a name="4-3"></a> 4.3 Status API
Messages sent to the status API consist of the same message content as queries. In addition, the identifier received as the query API’s response is transmitted through the [fin.020 submessage](#4-5).

Response messages are identical to the [query API's](#kyselyrajapinta-kysely) responses, plus: 
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

### <a name="4-6"></a> 4.6 Message extension Fin021 (QueryResultResponse)
The submessage schema is defined in the [fin.021](schemas/fin.021.001.01.xsd) file. 
The submessage is returned in the [data retrieval system’s response message](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#InformationRequestResponseV01) in the [ReturnIndicator1](#4-7) element similarly to other submessages.

|Name|[min..max]|Type|Description|Appended to message
|:---|:---|:---|:---|:---|
|QueryResultResponse| | | |[auth.002](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#InformationRequestResponseV01)|
|&nbsp;&nbsp;&nbsp;&nbsp;QueryKeyList|[1..1]|QueryKeyList|A list of identifiers used in the query||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKeyList|[1..1]|ResultKeyList]|A list of result identifiers||


### <a name="4-7"></a> 4.7 Use of the ReturnIndicator1 element with the fin.021 message

ReturnIndicator1 includes the presence of a single type of search result as in the [response messages](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#InformationRequestResponseV01) of the data retrieval system’s query API. The fin.021 submessage is returned in this element similarly to other submessages returned in the query.

|XPath|Type|Description|
|:---|:---|:---|
|RtrInd/AuthrtyReqTp/MsgNmId|Max35Text|Includes the message ID of the message extension (fin.021.001.01))|
|RtrInd/InvstgtnRslt|InvestigationResult1Choice|Returning `Rslt` element type SupplementaryDataEnvelope1, which includes the [QueryResultResponse](#QueryResultResponse) submessage or the `InvstgtnSts` element, useing code `NFOU`, if no information can be found using the identifier used in the query.

### <a name="4-8"></a> 4.8 Error management
Error management and returned codes follow the specifications of the [WS message traffic scenarios in the query API](https://finnishcustoms-suomentulli.github.io/account-register-information-query/index_en.html#4-12) chapter for the data retrieval system’s query API, where applicable.

### <a name="4-9"></a> 4.9 Example messages
Examples of each query message and their responses are available in the examples folder:
- [Query message](examples/query1.xml) and [response](examples/query1_response.xml)
- [Status query](examples/status1.xml) and [response](examples/status1_response.xml)
- [Result query](examples/result1.xml) and [response](examples/result1_response.xml)
  
