HUOM Tiedostot ja linkit niihin päivitettävä:  
[Käyttöönoton ja ylläpidon ohjeistus koostava sovellus](instructions/Käyttöönoton_ja_ylläpidon_ohjeistus_tiedonhakujärjestelmä.pdf)  
[Deployment and maintenance instructions for the Data Retrieval System](instructions/Deployment_and_maintenance_instructions_for_the_Data_Retrieval_System_EN.pdf)  
[Query interface description of the data retrieval system](index_en.md)  
[Instruktioner för produktionssättning och underhåll av datasöksystemet](instructions/Instruktioner_för_produktionssättning_och_underhåll_av_datasöksystemet_SV.pdf)  
[Beskrivning av datasöksystemets frågegränssnitt](index_sv.md)

# Koostavan sovelluksen kyselyrajapintakuvaus

*Dokumentin versio 1.0*

## Versiohistoria

Versio|Päivämäärä|Kuvaus
---|---|---
1.0|7.10.2022|Versio 1.0|


## Sisällysluettelo

1. [Johdanto](#luku1)  
2. [Pankki- ja maksutilitietojen kysely koostavasta sovelluksesta](#luku2)  
3. [Tietoturva](#tietoturva)  
4. [Koostavan sovelluksen kyselyrajapinta](#kyselyrajapinta)   
  4.1 [Kyselyrajapinnan SOAP-operaatioiden sanomarakenne](#4-1)    


## 1. Johdanto <a name="luku1"></a>

### 1.1 Termit ja lyhenteet

Lyhenne tai termi|Selite
---|---
Rajapinta|Standardin mukainen käytäntö tai yhtymäkohta, joka mahdollistaa tietojen siirron laitteiden, ohjelmien tai käyttäjän välillä. 
WS (Web Service)|Verkkopalvelimessa toimiva ohjelmisto, joka tarjoaa standardoitujen internetyhteyskäytäntöjen avulla palveluja sovellusten käytettäväksi. Tiedonhakujärjestelmä tarjoaa palveluna tietojen kyselyn.
Endpoint|Rajapintapalvelu, joka on saatavilla tietyssä verkko-osoitteessa
WSDL| (Web Service Description Language) Rakenteellinen kuvauskieli, jolla kuvataan web palvelun tarjoamat toiminnallisuudet.

### 1.2 Dokumentin tarkoitus ja kattavuus

Tämä dokumentti täydentää Tullin julkaisemaa määräystä pankki- ja maksutilien valvontajärjestelmästä. Dokumentin tarkoitus on antaa ohjeet koostavan sovelluksen kyselyrajapinnasta.

HUOM Onko osa määräystä kuten yllä sanotaan (kopioitu tiedonhakujärjestelmien dokumentista)
(E: ei liene osa määräystä, tullut käytännön tarpeesta)

### 1.3 Viittaukset

HUOM Kehitystiimin tarkastettava luvun sisältö

[Tiedonhakujärjestelmän WSDL](https://finnishcustoms-suomentulli.github.io/account-register-information-query/wsdl/data-retrieval-system-wsdl.xml)

[fin.020.001.01](schemas/fin.020.001.01.xsd)

[fin.021.001.01](schemas/fin.021.001.01.xsd)

[Sähköisen asioinnin tietoturvallisuus -ohje](http://julkaisut.valtioneuvosto.fi/bitstream/handle/10024/80012/VM_25_2017.pdf)

### 1.4 Yleiskuvaus

Tulli on perustanut Tilirekisterihankkeen, joka toteuttaa (EU) 2018/843 direktiivin ja sen täytäntöön panemiseksi säädetyn, Suomen lainsäädäntöön perustuvan pankki- ja maksutilien valvontajärjestelmän. 

Tässä dokumentissa kuvataan Koostavan sovelluksen rajapinnat.

## 2. Pankki- ja maksutilitietojen kysely koostavasta sovelluksesta <a name="luku2"></a>

Tässä luvussa on kuvattu pankki- ja maksutilitietojen kysely koostavasta sovelluksesta.

Kuvassa 2.1 kyselyprosessi on on esitetty vuokaaviona.

![Tietojen kysely koostavasta sovelluksesta](diagrams/flowchart_koostava.png "Tietojen kysely Koostavasta sovelluksesta")  
*__Kuva2.1.__ Tietojen kysely Koostavasta sovelluksesta*  

Taulukossa 2.1. on esitetty vuokaavion muuttujien merkitys. 

*__Taulukko 2.1.__ Vuokaavion muuttujat*

|Muuttuja|Kuvaus|
|:--|:--|
|POLLING_INTERVAL|Pollausväli, viive joka clientin on odotettava ennen seuraavaa kyselyä. Jos client pollaa serveriä liian tiheästi, voi server hylätä transaktion käsittelyn (virhekoodi 3, ks. [taulukko 4.12.1](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#4-12)).|
|POLLING_TIME_LIMIT|Kuinka kauan pollausta on sallittua tehdä, ennen kuin lopetetaan. Jos vastausta ei edelleenkään saada, on joko tehtävä kokonaan uusi kysely tai siirrettävä asia manuaaliseen käsittelyyn.|

Taulukossa 2.1. esitettyjen muuttujien kulloinkin voimassa olevat arvot kerrotaan liitedokumentaatiossa.

Kyselyn vuo kulkee seuraavasti:
1. Client lähettää kyselysanoman Query APIin.
2. Query API palauttaa vastauksena avaimen (resultKey).
3. Client lähettää avaimen sisältävän statuskyselyn Status APIin
4. Status API joko
  a. Palauttaa koodin NRES, jos tulokset eivät ole vielä valmiina, tai
  b. Palauttaa koodin COMP ja listan avain-tiedonlähde pareja.
5. Jos koodi on *NRES*, Client odottaa ja palaa kohtaan 3.
6. Jos koodi on *COMP*, Client lähettää hakutuloskyselyn yhdellä kohdassa 4.b. vastaanottamistaan avaimista Result APIin.
7. Result API palauttaa lähetettyä avainta vastaavan hakutulossanoman.
8. Jos hakutuloksia on vielä hakematta, palataan kohtaan 6.
9. Jos kaikki hakutulokset on saatu, loppu.
 
Taulukossa 2.2. on kuvattu StatusResponse1Code arvojen käyttö

*__Taulukko 2.2.__ StatusResponse1Code arvojen käyttö*

|Koodi|Nimi|Määritelmä|Kuvaus|
|:--|:--|:--|:--|
|COMP|CompleteResponse|Response is complete.|Vastaussanoma sisältää hakutulokset|
|NRES|NoResponseYet|Response not provided yet.|Vastaussanoma ei sisällä hakutuloksia, tee uusi kysely myöhemmin.|
|PART|PartialResponse|Response is partially provided.|Ei käytössä|

#### Tulosten säilytysaika Koostavassa sovelluksessa
Valmiita tuloksia pidetään tallessa korkeintaan 6h niiden valmistumisesta, jona aikana tulokset on noudettava. Tuloksia ei poisteta haettaessa, vaan vasta em. aikarajan umpeuduttua. 
(##TODO: määriteltävä aikaraja##)

## <a name="tietoturva"></a> 3. Tietoturva

Tietoturvakäytäntö noudattaa samoja periaatteita, kuin [Tiedonhakujärjestelmän kyselyrajapinnan](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#tietoturva) määrittelyssä on kuvattu.

## <a name="kyselyrajapinta"></a> 4. Koostavan sovelluksen rajapinta

Rajapinta toteutetaan SOAP/XML Web Servicenä, jonka [WSDL](https://finnishcustoms-suomentulli.github.io/account-register-information-query/wsdl/data-retrieval-system-wsdl.xml) on julkaistu Tiedonhakujärjestelmän kyselyrajapintakuvaus
-dokumentin yhteydessä.

SOAP-protokollasta käytetään versiota 1.1.

Sanomissa käytetään ISO 20022 koodistoviittauksia. Koodistoviittaukset löytyvät ISO 20022 sivulta [External Code Sets](https://finnishcustoms-suomentulli.github.io/account-register-information-query/assets/iso20022org/ExternalCodeSets_2Q2020_August2020_v1.xlsx).

Rajapinnassa käytettävä sanomarakenne on päätasolla identtinen Tullin julkaiseman [Tiedonhakujärjestelmien kyselyrajapinnan](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#kyselyrajapinta) määrityksen kanssa.
Tämän lisäksi rajapintaan on määritelty alisanomat [fin.020](#kyselyrajapinta-fin020) ja [fin.021](#kyselyrajapinta-fin021), joilla välitetään tarvittavat tunnisteet haun tilan tarkistamiseksi ja tulosten hakemiseksi.

Koostavan sovelluksen rajapinnassa on kolme endpointia: 
- [kyselyn vastaanottamiseen]("#kyselyrajapinta-kysely")
- [kyselyn statuksen kyselyyn]("#kyselyrajapinta-status")
- [hakutulosten noutoon]("#kyselyrajapinta-tulos")

### <a name="kyselyrajapinta-kysely"></a> 4.2 Kyselyrajapinta
Kyselyrajapintaan lähetettävä sanoma on täysin samanlainen kuin [Tiedonhakujärjestelmien kyselyrajapinnassa](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#kyselyrajapinta) käytettävä sanoma.

Vastaussanoma noudattaa myös samaa rakennetta, mutta auth.002-sanoman yhtenä alisanomana on fin.021-sanoma, joka kertoo vastaanotetun kyselyn tunnuksen. 
Tätä tunnusta käytetään kyselyn tilan tarkistamiseen [Status-rajapinnasta](#kyselyrajapinta-status). Muut alisanomat palauttavat statuksen NFOU, mikä ei tässä tapauksessa merkitse mitään.

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
Sanomalaajennus liitetään taulukossa listattuun ISO 20022 sanoman XPath-sijaintiin.

|Nimi|[min..max]|Tyyppi|Kuvaus|Liitetään sanomaan|XPath|
|:---|:---|:---|:---|:---|:---|
|QueryResultRequest| | | |[auth.001](#InformationRequestOpeningV01)|`/Document/InfReqOpng/SplmtryData/Envlp`|
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKeyList|[1..1]|[ResultKeyList](#ResultKeyList)|Lista haettavien tietojen tunnisteista||

### <a name="kyselyrajapinta-fin021"></a> 4.6 Sanomalaajennus Fin021 (QueryResultResponse)
Sanomalaajennus liitetään taulukossa listattuun ISO 20022 sanoman XPath-sijaintiin.

https://finnishcustoms-suomentulli.github.io/account-register-information-query/#InformationRequestResponseV01

|Nimi|[min..max]|Tyyppi|Kuvaus|Liitetään sanomaan
|:---|:---|:---|:---|:---|
|QueryResultResponse| | | |[auth.002](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#InformationRequestResponseV01)|
|&nbsp;&nbsp;&nbsp;&nbsp;QueryKeyList|[1..1]|[QueryKeyList](#QueryKeyList)|Lista kyselyssä käytetyistä tunnisteista||
|&nbsp;&nbsp;&nbsp;&nbsp;ResultKeyList|[1..1]|[ResultKeyList](#ResultKeyList)|Lista tulostietojen tunnisteista||


### <a name="kyselyrajapinta-rtrInd"></a> 4.7 Elementin ReturnIndicator1 käyttö fin.020- ja fin.021-sanomien kanssa

ReturnIndicator1 sisältää yksittäisen hakutulostyypin esiintymän, kuten Tiedonhakujärjestelmän kyselyrajapinnan [vastaussanomassakin](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#InformationRequestResponseV01).
fin.021 palautetaan tässä elementissä, kuten muutkin kyselyssä palautettavat alisanomat.

|XPath|Tyyppi|Kuvaus|
|:---|:---|:---|
|RtrInd/AuthrtyReqTp/MsgNmId|Max35Text|sisältää sanomalaajennuksen sanoma-id:n (fin.021.001.01)|
|RtrInd/InvstgtnRslt|InvestigationResult1Choice|palautetaan `Rslt` elementti tyyppiä SupplementaryDataEnvelope1, joka sisältää alisanoman [QueryResultResponse](#QueryResultResponse) tai elementin `InvstgtnSts` koodilla `NFOU`, jos kyselyssä käytetyllä tunnuksella ei löydy tietoa.

### <a name="kyselyrajapinta-virhetilanteet"></a> 4.8 Virhetilanteiden hallinta
Virheiden hallinta ja palautettavat koodit noudattavat soveltuvin osin Tiedonhakujärjestelmän kyselyrajapinnan [Kyselyrajapinnan WS-sanomaliikenteen skenaariot](https://finnishcustoms-suomentulli.github.io/account-register-information-query/#4-12) -kappaleen määrityksiä.

