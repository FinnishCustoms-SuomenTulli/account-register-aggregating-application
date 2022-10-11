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
  4.2 [Business Application Header (BAH)](#4-2)    
  4.3 [Kyselyrajapinnan sanomat](#4-3)    
  4.4 [BusinessApplicationHeaderV01](#BusinessApplicationHeaderV01)    
  4.5 [InformationRequestOpeningV01](#InformationRequestOpeningV01)    
  4.6 [InformationRequestFIN012](#InformationRequestFIN012)    
  4.7 [InformationRequestResponseV01](#InformationRequestResponseV01)    
  4.8 [InformationResponseSD1V01](#InformationResponseSD1V01)    
  4.9 [InformationResponseFIN002](#InformationResponseFIN002)    
  4.10 [InformationResponseFIN013](#InformationResponseFIN013)   
  4.11 [Id-elementin käyttö](#Id-elementin_kaytto)    
  4.12 [Kyselyrajapinnan WS-sanomaliikenteen skenaariot](#4-12)    
  4.13 [Kiistanalaisten tietojen palauttaminen](#4-13)  

## 1. Johdanto <a name="luku1"></a>

### 1.1 Termit ja lyhenteet

Lyhenne tai termi|Selite
---|---
Rajapinta|Standardin mukainen käytäntö tai yhtymäkohta, joka mahdollistaa tietojen siirron laitteiden, ohjelmien tai käyttäjän välillä. 
WS (Web Service)|Verkkopalvelimessa toimiva ohjelmisto, joka tarjoaa standardoitujen internetyhteyskäytäntöjen avulla palveluja sovellusten käytettäväksi. Tiedonhakujärjestelmä tarjoaa palveluna tietojen kyselyn.
Endpoint|Rajapintapalvelu, joka on saatavilla tietyssä verkko-osoitteessa
WSDL| (Web Service Description Language) Rakenteellinen kuvauskieli, jolla kuvataan web palvelun tarjoamat toiminnallisuudet.

### 1.2 Dokumentin tarkoitus ja kattavuus

Tämä dokumentti on osa Tullin julkaisemaa määräystä pankki- ja maksutilien valvontajärjestelmästä. Dokumentin tarkoitus on antaa ohjeet koostavan sovelluksen kyselyrajapinnasta. Tätä dokumenttia täydentää koostavan sovelluksen käyttöönoton ja ylläpidon ohje.

HUOM Onko osa määräystä kuten yllä sanotaan (kopioitu tiedonhakujärjestelmien dokumentista)

### 1.3 Viittaukset

HUOM Kehitystiimin tarkastettava luvun sisältö

[Tiedonhakujärjestelmän WSDL](wsdl/data-retrieval-system-wsdl.xml)

[ISO 20022 External Code Sets](assets/iso20022org/ExternalCodeSets_2Q2020_August2020_v1.xlsx)

[ISO 20022 auth.001.001.01 InformationRequestOpeningV01 MDR](assets/iso20022org/archive_business_area_authorities.zip)

[ISO 20022 auth.002.001.01 InformationRequestResponseV01 MDR](assets/iso20022org/archive_business_area_authorities.zip)

[ISO 20022 head.001.001.01 skeema](assets/iso20022org/archive_business_area_business_application_header.zip)

[fin.002.001.02](schemas/fin.002.001.02.xsd)

[fin.012.001.03](schemas/fin.012.001.03.xsd)

[fin.013.001.04](schemas/fin.013.001.04.xsd)

[Sähköisen asioinnin tietoturvallisuus -ohje](http://julkaisut.valtioneuvosto.fi/bitstream/handle/10024/80012/VM_25_2017.pdf)

### 1.4 Yleiskuvaus

Tulli on perustanut Tilirekisterihankkeen, joka toteuttaa (EU) 2018/843 direktiivin ja sen täytäntöön panemiseksi säädetyn, Suomen lainsäädäntöön perustuvan pankki- ja maksutilien valvontajärjestelmän. 

Tässä dokumentissa kuvataan koostavan sovelluksen rajapinnat.

## 2. Pankki- ja maksutilitietojen kysely koostavasta sovelluksesta <a name="luku2"></a>

Tässä luvussa on kuvattu pankki- ja maksutilitietojen kysely koostavasta sovelluksesta.

HUOM Onko lause tässä tarpen? Viranomaisten järjestelmissä on olemassa rajoituksia yksittäisen sanoman koolle.

HUOM linjattava aiheesta yhteinen linja, ei enää sovittavissa: Toimijat voivat käyttöönoton yhteydessä keskenään sopia sanomien kokorajasta sekä keinoista, millä tavoin yksittäisen sanoman kokoa voidaan tarvittaessa pienentää tai vaihtoehtoisesta menetelmästä satunnaisten ylisuurten sanomien toimittamiseen.

Kuvassa 2.1 on esitetty vuokaaviona pankki- ja maksutilitietojen kysely koostavasta sovelluksesta.

![Pankki- ja maksutilitietojen kysely](diagrams/flowchart_query.png "Pankki- ja maksutilitietojen kysely")  
*__Kuva2.1.__ Pankki- ja maksutilitietojen kysely*  

HUOM: Kuvasta nähdään, että kyselyrajapinta mahdollistaa sekä vastauksen palauttamisen heti synkronisesti, tai vaihtoehtoisesti asynkronisesti. 

Taulukossa 2.1. on esitetty vuokaavion muuttujien merkitys. 

*__Taulukko 2.1.__ Vuokaavion muuttujat*

|Muuttuja|Kuvaus|
|:--|:--|
|POLLING_INTERVAL|Pollausväli, viive joka clientin on odotettava ennen seuraavaa kyselyä. Jos client pollaa serveriä liian tiheästi, voi server hylätä transaktion käsittelyn (virhekoodi 3, ks [4.12](#4-12)).|
|RETRY_LIMIT|Kuinka monta pollausta on sallittua tehdä, ennen kuin lopetetaan. Jos vastausta ei edelleenkään saada, on joko tehtävä kokonaan uusi kysely tai siirrettävä asia manuaaliseen käsittelyyn.|

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

## <a name="tietoturva"></a> 3. Tietoturva

HUOM Koko 3. luku tarkastettava ja päivitettävä, kopioitu sellaisenaan tiedonhakujärjestelmien dokumentista

### 3.1 Tunnistaminen

Taulukossa 3.1. on esitetty varmenteet tiedonhakujärjestelmässä.

*__Taulukko 3.1.__ Tiedonhakujärjestelmän varmenteet*

|Standardi|Varmenteen nimi|Käyttötarkoitus|
|:--|:--|:--|
|X.509 (versio 3)|Tiedonhakujärjestelmän tietoliikennevarmenne|Rajapinnan hyödyntäjän ja tiedon luovuttajan tai tiedon luovuttajan valtuuttaman tahon tunnistaminen|
|X.509 (versio 3)|Tiedonhakujärjestelmän allekirjoitusvarmenne|Sanoman allekirjoittaminen, sanoman muuttumattomuuden varmistaminen, tiedon luovuttajan tunnistaminen|

Tiedonhakujärjestelmän kyselyrajapinnan hyödyntäjät sekä tiedon luovuttajat tai tiedon luovuttajan valtuuttamat tahot tunnistetaan X.509-varmenteilla (Tietoliikennevarmenne). Kyselyrajapinnan kysely- ja vastaussanomat allekirjoitetaan XML-allekirjoituksella (Allekirjoitusvarmenne).

#### Tiedon luovuttajan allekirjoitusvarmenne

Tiedon luovuttajan on allekirjoitettava lähettämänsä sanomat käyttäen x.509 palvelinvarmennetta, josta käy ilmi ko. tiedon luovuttajan Y-tunnus tai ALV-tunnus. Saapuvien sanomien allekirjoitus on tarkistettava. Vastaanottaja ei saa hyväksyä sanomaa ilman hyväksyttävää allekirjoitusta. Allekirjoituksen hyväksyminen edellyttää, että XML-allekirjoitus on validi ja että

joko  
a) varmenne on VRK:n myöntämä, voimassa, eikä esiinny VRK:n ylläpitämällä sulkulistalla ja varmenteen kohteen serialNumber attribuuttina on kyseisen tiedon luovuttajan Y-tunnus tai ALV-tunnus

tai  
b) varmenne on eIDAS-hyväksytty sivustojen tunnistamisvarmenne, voimassa, eikä esiinny varmenteen tarjoajan ylläpitämällä ajantasaisella sulkulistalla ja varmenteen kohteen organizationIdentifier-attribuuttina on kyseisen tiedon luovuttajan Y-tunnus tai ALV-tunnus.

Huom. Jotta sanomien allekirjoitukset täyttävät alla viitatut Kyberturvallisuuskeskuksen tietoturvavaatimukset, tulee allekirjoituksiin käytettävän varmenteen julkisen avaimen (RSA public key) olla vähintään 3072 bittinen. Allekirjoituksiin käytettävän varmenteen käyttötarkoituksiin myös kuulua ”digitaalinen allekirjoitus”. Nämä seikat tulee huomioida varmennetta tilattaessa.

#### Toimivaltaisen viranomaisen allekirjoitusvarmenne

Toimivaltaisen viranomaisen on allekirjoitettava lähettämänsä sanomat käyttäen x.509 palvelinvarmennetta, josta käy ilmi ko. viranomaisen Y-tunnus. Saapuvien sanomien allekirjoitus on tarkistettava. Vastaanottaja ei saa hyväksyä sanomaa ilman hyväksyttävää allekirjoitusta. Toimivaltaisen viranomaisen allekirjoituksen hyväksyminen edellyttää, että XML-allekirjoitus on validi ja että  
a) varmenne on VRK:n myöntämä, voimassa, eikä esiinny VRK:n ylläpitämällä sulkulistalla  
b) varmenteen kohteen serialNumber attribuuttina on sanoman lähettäneen toimivaltaisen viranomaisen Y-tunnus tai tunnus, joka muodostuu kirjaimista “FI” ja Y-tunnuksen numero-osasta ilman väliviivaa (ALV-tunnuksen muotoinen tunnus).

#### Yhteydenottajan tietoliikennevarmenne

Tiedon luovuttaja tai tiedon luovuttajan valtuuttama taho tunnistaa toimivaltaisen viranomaisen, joka ottaa yhteyden tiedonhakujärjestelmän kyselyrajapintaan, palvelinvarmenteen avulla. Yhteys toimivaltaiselta viranomaiselta on hyväksyttävä seuraavin edellytyksin: 
a) Toimivaltaisen viranomaisen varmenteen on myöntänyt VRK  
b) varmenne on voimassa, eikä esiinny VRK:n sulkulistalla  
c) varmenteen kohteen serialNumber attribuuttina on toimivaltaisen viranomaisen tai sen puolesta toimivan valtion palvelukeskuksen Y-tunnus tai tunnus, joka muodostuu kirjaimista “FI” ja Y-tunnuksen numero-osasta ilman väliviivaa (ALV-tunnuksen muotoinen tunnus).

#### Tiedon luovuttajan tai tiedon luovuttajan valtuuttaman tahon tietoliikennevarmenne

Toimivaltainen viranomainen, joka ottaa yhteyden kyselyrajapintaan, tunnistaa tiedon luovuttajan tai tiedon luovuttajan valtuuttaman tahon palvelinvarmenteen avulla. Tiedon luovuttajan valtuuttamalla taholla tarkoitetaan esim. palvelukeskusta, jonka tiedon luovuttaja on valtuuttanut puolestaan huolehtimaan ilmoitusten muodostamisesta ja/tai lähettämisestä.

Yhteys tiedon luovuttajaan on hyväksyttävä seuraavin edellytyksin:

joko  
a) palvelinvarmenteen on myöntänyt VRK, varmenne on voimassa, eikä esiinny VRK:n sulkulistalla, varmenteen kohteen serialNumber attribuuttina on kyseisen tiedon luovuttajan tai tiedon luovuttajan valtuuttaman tahon Y-tunnus tai ALV-tunnus

tai  
b) palvelinvarmenne on eIDAS-hyväksytty sivustojen tunnistamisvarmenne, voimassa, eikä esiinny varmenteen tarjoajan ylläpitämällä ajantasaisella sulkulistalla ja varmenteen kohteen organizationIdentifier-attribuuttina on kyseisen tiedon luovuttajan tai tiedon luovuttajan valtuuttaman tahon Y-tunnus tai ALV-tunnus.

Mikäli tiedon luovuttajan tietoliikennevarmenteessa ja lähtevän sanoman allekirjoitusvarmenteessa käytetään samaa Y-tunnusta tai ALV-tunnusta, voidaan kumpaankin tarkoitukseen käyttää samaa varmennetta.

Huom. Jotta tietoliikenteen suojaus täyttää alla viitatut Kyberturvallisuuskeskuksen tietoturvavaatimukset, tulee käytettävän varmenteen julkisen avaimen (RSA public key) olla vähintään 3072 bittinen. Tämä tulee huomioida varmennetta tilattaessa.

#### <a name="xml-sig"></a> XML-allekirjoituksen muodostaminen

Allekirjoituksen tyyppi on __enveloped signature__. Signature-elementti sijoitetaan [BAHin](#BusinessApplicationHeaderV01) Sgntr-elementin alle.

Esimerkki 3.1. Esimerkki SignedInfo
```
<SignedInfo>
  <CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
  <SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />
  <Reference URI="#applicationRequest">
    <Transforms>
      <Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />
      <Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
    </Transforms>
    <DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256" />
    <DigestValue />
  </Reference>
</SignedInfo>
```

Allekirjoitusalgoritmi on siis RSA-SHA256 ja C14N on Exclusive XML Canonicalization. Reference URI on joko "#applicationRequest" tai "#applicationResponse", riippuen onko kyseessä kysely- vai vastaussanoma. Eli vain "ApplicationRequest" tai "ApplicationResponse" elementti allekirjoitetaan. Allekirjoitusta muodostettaessa laskettavien tiivisteiden (Digest) muodostamiseen tulee käyttää SHA256 -algoritmia.

Allekirjoituksessa käytettyjen kryptografisten algoritmien on vastattava kryptografiselta vahvuudeltaan vähintään Viestintäviraston määrittelemiä kryptografisia vahvuusvaatimuksia kansalliselle suojaustasolle ST IV. Tämänhetkiset vahvuusvaatimukset on kuvattu dokumentissa https://www.kyberturvallisuuskeskus.fi/sites/default/files/media/regulation/ohje-kryptografiset-vahvuusvaatimukset-kansalliset-suojaustasot.pdf (Dnro: 190/651/2015).

Mahdollisuus pyyntöjen IP-avaruuden rajoittamiseen tiedonhakujärjestelmässä tarkennetaan myöhemmin.

### 3.2 Yhteyksien suojaaminen

Tiedonhakujärjestelmän kyselyrajapinnan yhteydet on suojattava TLS-salauksella käyttäen TLS-protokollan versiota 1.2 tai korkeampaa. Yhteyden molemmat päät tunnistetaan yllä kuvatuilla palvelinvarmenteilla käyttäen kaksisuuntaista kättelyä. Yhteys on muodostettava käyttäen ephemeral Diffie-Hellman (DHE) avaintenvaihtoa, jossa jokaiselle sessiolle luodaan uusi uniikki yksityinen salausavain. Tämän menettelyn tarkoituksena on taata salaukselle forward secrecy -ominaisuus, jotta salausavaimen paljastuminen ei jälkikäteen johtaisi salattujen tietojen paljastumiseen.

TLS-salauksessa käytettyjen kryptografisten algoritmien on vastattava kryptografiselta vahvuudeltaan vähintään Viestintäviraston määrittelemiä kryptografisia vahvuusvaatimuksia kansalliselle suojaustasolle ST IV. Tämänhetkiset vahvuusvaatimukset on kuvattu dokumentissa https://www.kyberturvallisuuskeskus.fi/sites/default/files/media/regulation/ohje-kryptografiset-vahvuusvaatimukset-kansalliset-suojaustasot.pdf (Dnro: 190/651/2015).

### 3.3 Sallittu HTTP-versio

Tiedonhakujärjestelmän kyselyrajapinnan yhteydet käyttävät HTTP-protokollan versiota 1.1.

### 3.4 Tietoturvapoikkeamien ilmoitusvelvollisuus

Mikäli tiedonhakujärjestelmän toteuttajan varmenteet tai näiden salaiset avaimet vaarantuvat on tästä ilmoitettava välittömästi varmenteen myöntäjälle ja niille toimivaltaisille viranomaisille, jotka hyödyntävät tiedonhakujärjestelmää. Toimivaltaisille viranomaisille on ilmoitettava myös, jos tiedonhakujärjestelmässä havaitaan tietoturvapoikkeama

Mikäli tiedonhakujärjestelmää hyödyntävän toimivaltaisen viranomaisen varmenteet tai näiden salaiset avaimet vaarantuvat on tästä ilmoitettava välittömästi varmenteen myöntäjälle ja niille tiedonhakujärjestelmän toteuttajille, joiden toteutusta tiedonhakujärjestelmästä kyseinen toimivaltainen viranomainen hyödyntää.

## <a name="kyselyrajapinta"></a> 4. Koostavan sovelluksen rajapinta

Rajapinta toteutetaan SOAP/XML Web Servicenä, josta julkaistaan [WSDL](wsdl/data-retrieval-system-wsdl.xml).

SOAP-protokollasta käytetään versiota 1.1.

Sanomissa käytetään ISO 20022 koodistoviittauksia. Koodistoviittaukset löytyvät ISO 20022 sivulta [External Code Sets](assets/iso20022org/ExternalCodeSets_2Q2020_August2020_v1.xlsx).

Koostavan sovelluksen rajapinnassa on kolme endpointia: haun vastaanottamiseen, haun statuksen kyselyyn ja hakutulosten noutoon.
...jonka kysely- ja vastaussanomarakenne on kuvattu tässä luvussa.
