[Tiedon hyödyntäjien käyttöönoton ja ylläpidon ohje, koostava sovellus](instructions/Tiedon_hyödyntäjien_käyttöönoton_ja_ylläpidon_ohje_koostava_sovellus.pdf)  
[Deployment and maintenance instructions for data users, Aggregating Application](instructions/Deployment_and_maintenance_instructions_for_data_users_Aggregating_Application.pdf)  
[Description of the aggregating application’s query API](index_en.md)  
[Instruktioner för uppgiftsanvändare om ibruktagande och underhåll, sammanställningsprogrammet](instructions/Instruktioner_för_uppgiftsanvändare_om_ibruktagande_och_underhåll_sammanställningsprogrammet.pdf)  
[Beskrivning av sammanställningsprogrammets frågegränssnitt](index_sv.md)  

# Koostavan sovelluksen rajapintakuvaus

*Dokumentin versio 1.01*

## Versiohistoria

| Versio | Päivämäärä | Kuvaus                                                              |
|--------|------------|---------------------------------------------------------------------|
| 1.0    | 7.2.2023   | Versio 1.0                                                          |
| 1.01   | 28.6.2023   | Päivitetty lukuun 4.2 kaksi kyselyssä käytettävää uutta tietokenttää. Toista käytetään kohdistamaan kysely tiety(i)lle tiedonlähteille, toista merkitsemään kyselyn liittyvän kansainväliseen/rajat ylittävään tietopyyntöön. Lisäksi päivitetty lukuun 4.7 InvstgtnSts NOAP käyttö vastaussanomassa.|

## Sisällysluettelo

1. [Johdanto](#luku1)  
2. [Pankki- ja maksutilitietojen kysely koostavasta sovelluksesta](#luku2)  
3. [Tietoturva](#tietoturva)  
4. [Koostavan sovelluksen kyselyrajapinta](#kyselyrajapinta)   
  4.1 [Kyselyrajapinnan SOAP-operaatioiden sanomarakenne](#kyselyrajapinta-rakenne)    
  4.2 [Kyselyrajapinta](#kyselyrajapinta-kysely)    
  4.3 [Status-rajapinta](#kyselyrajapinta-status)    
  4.4 [Tulosrajapinta](#kyselyrajapinta-tulos)    
  4.5 [Sanomalaajennus Fin020 (QueryResultRequest)](#kyselyrajapinta-fin020)    
  4.6 [Sanomalaajennus Fin021 (QueryResultResponse)](#kyselyrajapinta-fin021)    
  4.7 [Elementin ReturnIndicator1 käyttö](#kyselyrajapinta-rtrInd)    
  4.8 [Virhetilanteiden hallinta](#kyselyrajapinta-virhetilanteet)    
  4.9 [Esimerkkisanomat](#kyselyrajapinta-esimerkit)    
  

## 1. Johdanto <a name="luku1"></a>

### 1.1 Termit ja lyhenteet

| Lyhenne tai termi | Selite                                                                                                                                                                                              |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Rajapinta         | Standardin mukainen käytäntö tai yhtymäkohta, joka mahdollistaa tietojen siirron laitteiden, ohjelmien tai käyttäjän välillä.                                                                       |
| WS (Web Service)  | Verkkopalvelimessa toimiva ohjelmisto, joka tarjoaa standardoitujen internetyhteyskäytäntöjen avulla palveluja sovellusten käytettäväksi. Tiedonhakujärjestelmä tarjoaa palveluna tietojen kyselyn. |
| Endpoint          | Rajapintapalvelu, joka on saatavilla tietyssä verkko-osoitteessa                                                                                                                                    |
| WSDL              | (Web Service Description Language) Rakenteellinen kuvauskieli, jolla kuvataan web palvelun tarjoamat toiminnallisuudet.                                                                             |

### 1.2 Dokumentin tarkoitus ja kattavuus

Tämä dokumentti täydentää Tullin julkaisemaa määräystä pankki- ja maksutilien valvontajärjestelmästä. Dokumentin tarkoitus on antaa ohjeet koostavan sovelluksen rajapinnasta ja sen käytöstä. Tätä dokumenttia täydentää tiedon hyödyntäjän käyttöönoton ja ylläpidon ohje.

### 1.3 Viittaukset

[Tiedonhakujärjestelmän WSDL](https://finnishcustoms-suomentulli.github.io/account-register-information-query/wsdl/data-retrieval-system-wsdl.xml)

[fin.020.001.01](schemas/fin.020.001.01.xsd)

[fin.021.001.03](schemas/fin.021.001.03.xsd)

[fin.012.001.03](schemas/fin.012.001.03.xsd)

[Sähköisen asioinnin tietoturvallisuus -ohje](http://julkaisut.valtioneuvosto.fi/bitstream/handle/10024/80012/VM_25_2017.pdf)

### 1.4 Yleiskuvaus

Tulli on perustanut Tilirekisterihankkeen, joka toteuttaa (EU) 2018/843 direktiivin ja sen täytäntöön panemiseksi säädetyn, Suomen lainsäädäntöön perustuvan pankki- ja maksutilien valvontajärjestelmän. 

Tässä dokumentissa kuvataan Koostavan sovelluksen rajapinnat.

## 2. Pankki- ja maksutilitietojen kysely koostavasta sovelluksesta <a name="luku2"></a>

Tässä luvussa on kuvattu pankki- ja maksutilitietojen kysely koostavasta sovelluksesta.

Kuvassa 2.1 kyselyprosessi on on esitetty vuokaaviona.

![Tietojen kysely koostavasta sovelluksesta](diagrams/flowchart_koostava.png "Tietojen kysely Koostavasta sovelluksesta")  
*__Kuva 2.1.__ Tietojen kysely Koostavasta sovelluksesta*  

Taulukossa 2.1. on esitetty vuokaavioon liittyvien muuttujien merkitys. 

*__Taulukko 2.1.__ Vuokaavioon liittyvät muuttujat*

| Muuttuja           | Kuvaus                                                                                                                                                                                                                                                                                                     |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| POLLING_INTERVAL   | Pollausväli eli viive joka clientin on odotettava ennen seuraavaa kyselyä on 1 minuutti. Jos client pollaa serveriä liian tiheästi, voi server hylätä transaktion käsittelyn (virhekoodi 3, ks. [taulukko 4.12.1](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#4-12)). |
| POLLING_TIME_LIMIT | Kuinka kauan pollausta on sallittua tehdä, ennen kuin lopetetaan. Jos vastausta ei edelleenkään saada, on joko tehtävä kokonaan uusi kysely tai siirrettävä asia manuaaliseen käsittelyyn.                                                                                                                 |

Kyselyn vuo kulkee seuraavasti:
1. Client lähettää kyselysanoman Query API:in.
2. Query API palauttaa vastauksena avaimen (resultKey).
3. Client odottaa hetken (kts. POLLING_INTERVAL) ja lähettää avaimen sisältävän statuskyselyn Status APIin
4. Status API joko  
  a. Palauttaa koodin NRES, jos tulokset eivät ole vielä valmiina, tai  
  b. Palauttaa koodin COMP ja listan avaimia. 
5. Jos koodi on *NRES*, Client palaa kohtaan 3.
6. Jos koodi on *COMP*, Client lähettää hakutuloskyselyn yhdellä kohdassa 4.b. vastaanottamistaan avaimista Result APIin.
7. Result API palauttaa lähetettyä avainta vastaavan hakutulossanoman.
8. Jos hakutuloksia on vielä hakematta, palataan kohtaan 6 ja toistetaan haku seuraavalla avaimella.
9. Jos kaikki hakutulokset on saatu, loppu.
 
Palautettavat koodit on määritelty ISO-koodistossa StatusResponse1Code, jonka arvojen käyttön on kuvattu Taulukossa 2.2.

*__Taulukko 2.2.__ StatusResponse1Code arvojen käyttö*

|Koodi|Nimi|Määritelmä| Kuvaus                                                            |
|:---|:---|:---|:---|
|COMP|CompleteResponse|Response is complete.| Vastaussanoma sisältää hakutulokset                               |
|NRES|NoResponseYet|Response not provided yet.| Vastaussanoma ei sisällä hakutuloksia, tee uusi kysely myöhemmin. |
|PART|PartialResponse|Response is partially provided.| Ei käytössä                                                       |

#### Tulosten säilytysaika Koostavassa sovelluksessa
Tulokset poistetaan kun ne on noudettu. Valmiita tuloksia pidetään tallessa korkeintaan 24 tunnin ajan niiden valmistumisesta, jona aikana tulokset on noudettava. 

## <a name="tietoturva"></a> 3. Tietoturva

Tietoturvakäytäntö noudattaa samoja periaatteita, kuin [Tiedonhakujärjestelmän kyselyrajapinnan](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#tietoturva) määrittelyssä on kuvattu.

## <a name="kyselyrajapinta"></a> 4. Koostavan sovelluksen rajapinta

Rajapinta toteutetaan SOAP/XML Web Servicenä, jonka [WSDL](https://finnishcustoms-suomentulli.github.io/account-register-information-query/wsdl/data-retrieval-system-wsdl.xml) on julkaistu Tiedonhakujärjestelmän kyselyrajapintakuvaus
-dokumentin yhteydessä.

SOAP-protokollasta käytetään versiota 1.1.

Sanomissa käytetään ISO 20022 koodistoviittauksia. Koodistoviittaukset löytyvät ISO 20022 sivulta [External Code Sets](https://finnishcustoms-suomentulli.github.io/account-register-information-query/assets/iso20022org/ExternalCodeSets_2Q2020_August2020_v1.xlsx).

Koostavan sovelluksen rajapinnassa on kolme endpointia: 
- [kyselyn vastaanottamiseen](#kyselyrajapinta-kysely)
- [kyselyn statuksen kyselyyn](#kyselyrajapinta-status)
- [hakutulosten noutoon](#kyselyrajapinta-tulos)

### <a name="kyselyrajapinta-rakenne"></a> 4.1 Kyselyrajapinnan SOAP-operaatioiden sanomarakenne
Rajapinnassa käytettävä sanomarakenne on päätasolla identtinen Tullin julkaiseman [Tiedonhakujärjestelmien kyselyrajapinnan](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#kyselyrajapinta) määrityksen kanssa.
Tämän lisäksi rajapintaan on määritelty alisanomat [fin.020](#kyselyrajapinta-fin020) ja [fin.021](#kyselyrajapinta-fin021), joilla välitetään tarvittavat tunnisteet haun tilan tarkistamiseksi ja tulosten hakemiseksi.

### <a name="kyselyrajapinta-kysely"></a> 4.2 Kyselyrajapinta
Kyselyrajapintaan lähetettävä sanoma on enimmäkseen samanlainen kuin [Tiedonhakujärjestelmien kyselyrajapinnassa](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#kyselyrajapinta) käytettävä sanoma. Erona tiedonhakujärjestelmien kyselyrajapinnassa käytettävään sanomaan, fin012 sanomalaajennuksessa on käytettävissä kaksi lisäkenttää: RequestedDataSources ja InternationalRequest. RequestedDataSources-kenttää käytetään kohdistamaan kysely yhteen tai useampaan tiedonlähteeseen (tiedonhakujärjestemät ja tilirekisteri). Kohdistettu kysely välitetään ainoastaan kentässä määritellyille tiedonlähteille. InternationalRequest-kenttää käyttäen merkitään kyselyn liittyvän kansainväliseen/rajat ylittävään tietopyyntöön rahoitustietodirektiiviin ((EU) 2019/1153) 19 artiklaan ja kansallisen rahanpesulain (444/2017) 2 luvun 4 §:ään perustuen.

Vastaussanoma noudattaa myös samaa rakennetta, mutta auth.002-sanoman yhtenä alisanomana on fin.021-sanoma, joka kertoo vastaanotetun kyselyn tunnuksen. Tätä tunnusta käytetään kyselyn tilan tarkistamiseen [Status-rajapinnasta](#kyselyrajapinta-status). Status-rajapinnassa muut alisanomat palauttavat statuksen NFOU, mikä ei tässä tapauksessa merkitse mitään.

#### <a name="fin012"></a> 4.2.1 Sanomalaajennus InformationRequestFIN012
Alisanoman skeema on määritelty tiedostossa [fin.012](schemas/fin.012.001.03.xsd).
Sanomalaajennus liitetään taulukossa listattuun ISO 20022 sanoman XPath-sijaintiin.

| Nimi                                             | [min..max] | Tyyppi                                         | Kuvaus                                                                                                                                | Liitetään sanomaan | XPath                                    |
|:-------------------------------------------------|:-----------|:-----------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------|:-------------------|:-----------------------------------------|
| InformationRequestFIN012                         |            |                                                |                                                                                                                                       | auth.001           | `/Document/InfReqOpng/SplmtryData/Envlp` |
| &nbsp;&nbsp;&nbsp;&nbsp;AuthorityInquiry         | [1..1]     | [AuthorityInquirySet](#authority-inquiry-set)  | Kyselyyn liittyvät viranomaisen tiedot                                                                                                |                    |
| &nbsp;&nbsp;&nbsp;&nbsp;AdditionalSearchCriteria | [0..\*]    |                                                | Käytetään hakuun tallelokeron tunnisteella.                                                                                           |                    |
| &nbsp;&nbsp;&nbsp;&nbsp;RequestedDataSources     | [0..1]     | [RequestedDataSources](#requested-datasources) | Tiedonlähde tai tiedonlähteet, joille kysely lähetetään. Jos elementti puuttuu sanomasta, kysely lähetetään kaikille tiedonlähteille. |                    |
| &nbsp;&nbsp;&nbsp;&nbsp;InternationalRequest     | [0..1]     | boolean                                        | Käytetään arvoa "true", kun kysely liittyy kansainväliseen tietopyyntöön.                                                             |                    |

#### <a name="authority-inquiry-set"></a> AuthorityInquirySet

| Nimi                                       | Tyyppi     | Käytössä | Kuvaus                                          |
|:-------------------------------------------|:-----------|:---------|:------------------------------------------------|
| AuthorityInquirySet                        |            |          |                                                 |
| &nbsp;&nbsp;&nbsp;&nbsp;OfficialId         | Max140Text | Kyllä    | Kyselyn tehneen viranomaisen tunnus             |
| &nbsp;&nbsp;&nbsp;&nbsp;OfficialSuperiorId | Max140Text | Kyllä    | Kyselyn tehneen viranomaisen esihenkilön tunnus |

#### <a name="requested-datasources"></a> RequestedDataSources

| Nimi                                    | [min..max] | Tyyppi    | Kuvaus                 |
|:----------------------------------------|:-----------|:----------|:-----------------------|
| RequestedDataSources                    |            |           |                        |
| &nbsp;&nbsp;&nbsp;&nbsp;DataSourceOrgId | [1..\*]    | Max35Text | Tiedonlähteen Y-tunnus |

### <a name="kyselyrajapinta-status"></a> 4.3 Status-rajapinta
Status-rajapintaan lähetettävä sanoma koostuu samasta sanomasisällöstä kuin kyselyä tehtäessä, lisäksi kyselyrajapinnan vastauksena saatu tunnus välitetään [fin.020-alisanomassa](#kyselyrajapinta-fin020).

Vastaussanoma on kuten [Kyselyrajapinnan](#kyselyrajapinta-kysely) vastauksessa, ja lisäksi: 
- Mikäli kysely ei ole vielä valmistunut, palautuu Auth.002-skeeman RspnSts-kentässä tilakoodina NRES.
- Mikäli kysely on valmistunut, palautuu Auth.002-skeeman RspnSts-kentässä tilakoodina COMP ja lisäksi [fin.021-alisanomassa](#kyselyrajapinta-fin021) Status-kyselyssä käytetty tunnus sekä lista tunnuksista, joilla tulokset voi noutaa [Tulosrajapinnasta](#kyselyrajapinta-tulos). 

### <a name="kyselyrajapinta-tulos"></a> 4.4 Tulosrajapinta
Tulosrajapintaan lähetettävä sanoma koostuu samasta sanomasisällöstä kuin kyselyä tehtäessä, lisäksi Status-rajapinnan vastauksena saadut tunnukset välitetään yksi kerrallaan [fin.020-alisanomassa](#kyselyrajapinta-fin020).

Vastaussanoma on samanlainen, kuin Tiedonhakujärjestelmän kyselyrajapinnan [vastaussanoma](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#InformationRequestResponseV01), sisältäen varsinaiset hakutulokset skeeman mukaisesti.
Lisäksi [fin.021-alisanomassa](#kyselyrajapinta-fin021) palautuu tuloksen tunnus sekä _datasourceIdentifier_-attribuutissa tuloksen palauttaneen tiedonhakujärjestelmän Y-tunnus.


### <a name="kyselyrajapinta-fin020"></a> 4.5 Sanomalaajennus Fin020 (QueryResultRequest)
Alisanoman skeema on määritelty tiedostossa [fin.020](schemas/fin.020.001.01.xsd).
Sanomalaajennus liitetään taulukossa listattuun ISO 20022 sanoman XPath-sijaintiin.

|Nimi|[min..max]|Tyyppi|Kuvaus|Liitetään sanomaan|XPath|
|:---|:---|:---|:---|:---|:---|
|QueryResultRequest| | | |[auth.001](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#InformationRequestOpeningV01)|`/Document/InfReqOpng/SplmtryData/Envlp`|
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKeyList|[1..1]|ResultKeyList|Lista haettavien tietojen tunnisteista||

|Nimi|[min..max]|Tyyppi|Kuvaus|Huomioita|
|:---|:---|:---|:---|:---|
|ResultKeyList| | | ||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKey|[1..n]|Max256Text|Haun tai tuloksen UUID|Toistaiseksi tuetaan vain yhtä arvoa kerrallaan|

### <a name="kyselyrajapinta-fin021"></a> 4.6 Sanomalaajennus Fin021 (QueryResultResponse)
Alisanoman skeema on määritelty tiedostossa [fin.021](schemas/fin.021.001.03.xsd).
Alisanoma palautetaan [Tiedonhakujärjestelmän vastaussanomassa](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#InformationRequestResponseV01)
muiden alisanomien tapaan [ReturnIndicator1](#kyselyrajapinta-rtrInd)-elementin sisällä.

ResultKeyList-elementin attribuutteina palautetaan seuraavat tiedot:
- datasourceOrganisationId: tiedonlähteen Y-tunnus tai ALV-tunnus
- errorCode: virheen sattuessa tiedonlähteen palauttama tai Koostavan sovelluksen luoma virhekoodi, kts. kohta [4.8 Virhetilanteiden hallinta](#kyselyrajapinta-virhetilanteet)

|Nimi|[min..max]|Tyyppi|Kuvaus|Liitetään sanomaan|XPath|
|:---|:---|:---|:---|:---|:---|
|QueryResultResponse| | | |[auth.002](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#InformationRequestResponseV01)|`/Document/InfReqRspn/RtrInd/InvstgtnRslt/Rslt`|
|&nbsp;&nbsp;&nbsp;&nbsp;QueryKeyList|[0..1]|QueryKeyList|Lista kyselyssä käytetyistä tunnisteista||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKeyList|[0..1]|ResultKeyList|Lista tulostietojen tunnisteista||


|Nimi|[min..max]|Tyyppi| Kuvaus         |Huomioita|
|:---|:---|:---|:---------------|:---|
|QueryKeyList| | |                ||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKey|[1..n]|Max256Text| Tuloksen UUID  ||
|ResultKeyList| | |                ||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKey|[1..n]|Max256TextAllowedEmpty| Tuloksen UUID tai virhetilanteessa tyhjä |Optionaaliset attribuutit: `datasourceOrganisationId` ja `errorCode`|


### <a name="kyselyrajapinta-rtrInd"></a> 4.7 Elementin ReturnIndicator1 käyttö

ReturnIndicator1 sisältää yksittäisen hakutulostyypin esiintymän, kuten Tiedonhakujärjestelmän kyselyrajapinnan [vastaussanomassakin](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#InformationRequestResponseV01).
fin.021 palautetaan tässä elementissä, kuten muutkin kyselyssä palautettavat alisanomat.

| XPath                       | Tyyppi                     | Kuvaus                                                                                                                                                                                                                                      |
|:----------------------------|:---------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| RtrInd/AuthrtyReqTp/MsgNmId | Max35Text                  | sisältää sanomalaajennuksen sanoma-id:n (fin.021.001.03)                                                                                                                                                                                    |
| RtrInd/InvstgtnRslt         | InvestigationResult1Choice | Palautetaan `Rslt` elementti tyyppiä SupplementaryDataEnvelope1, joka sisältää jonkin alisanomista ([fin.021](#kyselyrajapinta-fin021), supl.027, fin.013 tai fin.002) tai elementin `InvstgtnSts`. `InvstgtnSts` palautetaan koodilla `NFOU`, jos kyselyssä käytetyllä tunnuksella ei löydy alisanomassa palautettavaa tietoa. `InvstgtnSts` palautetaan koodilla `NOAP` siinä tapauksessa, että luonnollisen tai oikeushenkilön nimihaulla tilirekisteristä löytyy jonkin toimijan tiedoista useita hakua vastaavia luonnollisia tai oikeushenkilöitä.|

### <a name="kyselyrajapinta-virhetilanteet"></a> 4.8 Virhetilanteiden hallinta
Virheiden hallinta ja palautettavat koodit noudattavat soveltuvin osin Tiedonhakujärjestelmän kyselyrajapinnan [Kyselyrajapinnan WS-sanomaliikenteen skenaariot](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#4-12) -kappaleen määrityksiä.

Status-rajapinnan vastaus voi olla tyhjä ResultKey ja virhekoodi, mikä tarkoittaa että kyselyn käsittelyssä on tapahtunut virhe. Tällöin kyselyä ei ole välitetty tietolähteisiin eikä kyselyyn tule noudettavia vastauksia. Tällä hetkellä "TIMEOUT" on ainoa virhekoodi, joka palautetaan tässä tilanteessa. 

Status- ja tulosrajapinnoista palautuvissa vastaussanomissa fin021-alisanomassa on myös tieto mahdollisesta virheestä, joka on tapahtunut yksittäisen tiedonlähteen osalla. Tämä tieto välitetään ResultKey-elementin `errorCode`-attribuutissa.

Koostavan sovelluksen kautta ei ole mahdollista kysellä 1.9.2020 aikaisempia tietoja. Koostavan sovelluksen kyselyrajapinta hylkää kyselyt, joissa haettu aikaväli (InvstgtnPrd) on ennen tätä.

### <a name="kyselyrajapinta-esimerkit"></a> 4.9 Esimerkkisanomat
Esimerkkisanomat kustakin kyselysanomasta ja sen vastauksesta löytyvät examples-kansiosta:
- [Kyselysanoma](examples/query1.xml) ja [vastaus](examples/query1_response.xml)
- [Status-kysely](examples/status1.xml), [vastaus](examples/status1_response.xml) ja [virheilmoitus](examples/status1_response_with_query_error.xml).
- [Tulos-kysely](examples/result1.xml), [vastaus](examples/result1_response.xml) ja [vastaus kyselyn kohdistuksella ja kansainvälisellä tietopyynnöllä](examples/query_with_requested_datasources_and_international_request.xml).
  
