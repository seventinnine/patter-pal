# PatterPal - Personal Language Teacher
*This project is a submission for the [Microsoft AI Classroom Hackathon](https://microsoftaiclassroom.devpost.com/).*

![PatterPal Screenshot](docs/img/uimain.PNG)

Immerse yourself in language learning: Talk freely on your chosen topics, get live feedback, and engage with a smart conversation partner in multiple languages. Learn your way with PatterPal!

## How it works
PatterPal is a web application that allows you to practice your speaking skills in a foreign language. *It's like having a language teacher just for yourself.*  PatterPal will listen to you and give you feedback on your pronunciation and fluency, while also talking with you about your chosen topic.

## Technologies
- Azure Speech Services
  - Speech to Text
  - Pronounciation Assessment
  - Synthesis
- Azure App Services
  - ASP.NET Core Web App
- Azure Cosmos DB
- Google OAuth 2.0
- OpenAi
  - gpt-3.5-turbo
  
<p>&nbsp;</p>
<div align="center">
	<code><img width="64" src="https://user-images.githubusercontent.com/25181517/121405384-444d7300-c95d-11eb-959f-913020d3bf90.png" alt="C#" title="C#"/></code>
	<code><img width="64" src="https://user-images.githubusercontent.com/25181517/183911544-95ad6ba7-09bf-4040-ac44-0adafedb9616.png" alt="Microsoft Azure" title="Microsoft Azure"/></code>
	<code><img width="64" src="https://user-images.githubusercontent.com/25181517/187070862-03888f18-2e63-4332-95fb-3ba4f2708e59.png" alt="websocket" title="websocket"/></code>
	<code><img width="64" src="https://user-images.githubusercontent.com/25181517/192107858-fe19f043-c502-4009-8c47-476fc89718ad.png" alt="REST" title="REST"/></code>
	<code><img width="64" src="https://user-images.githubusercontent.com/25181517/117447155-6a868a00-af3d-11eb-9cfe-245df15c9f3f.png" alt="JavaScript" title="JavaScript"/></code>
	<code><img width="64" src="https://user-images.githubusercontent.com/25181517/192158954-f88b5814-d510-4564-b285-dff7d6400dad.png" alt="HTML" title="HTML"/></code>
	<code><img width="64" src="https://user-images.githubusercontent.com/25181517/183898674-75a4a1b1-f960-4ea9-abcb-637170a00a75.png" alt="CSS" title="CSS"/></code>
	<code><img width="64" src="https://user-images.githubusercontent.com/25181517/192108374-8da61ba1-99ec-41d7-80b8-fb2f7c0a4948.png" alt="GitHub" title="GitHub"/></code>
    <code><img alt="PatterPal" title="PatterPal" width="64" height="64" src="docs/img/logo.svg"/></code>
</div>

## CLC3 Questions
This section will be written in German for the course CLC3.

### Automated Infrastructue Provisioning/(Infrastructure-as-Code). Wie wurde im vorliegenden Projekt Automated Infrastructure Provisioning berücksichtigt?
Die Azure Ressourcen-Gruppen-Konfiguration kann jederzeit exportiert und importiert werden. Dies erlaubt ein einfaches Neuaufsetzen des Projekts auch auf verschiedenen Azure Accounts.

**Azure App Services:**
Das Projekt wird durch eine automatisierte GitHub-Actions-Pipeline bei Merges auf Production auf den Azure App Service geladen. 

### Skalierbarkeit. Wie wurde im vorliegenden Projekt Skalierbarkeit berücksichtigt?

**Azure App Services:**
Implementiertes Backend ist zustandslos => mehrfaches Deployment + Load-Balancing ist möglich. Scale up & out.

**Cosmos DB:**
Würde auch mehrere Methoden zur Skalierung anbieten. Die Request Units (RUs) könnten theoretisch automatisch nach Auslastung skaliert werden.

### Ausfallssicherheit. Wie wurde im vorliegenden Projekt Ausfallssicherheit berücksichtigt?

**Azure App Services:**
Kein SLA im Free-Tier, bei B1 99.95% Availability. Die App kann auch in verschiedenen Regionen Deployments haben.

**Cosmos DB:**
Free Tier garantiert eine 99.99% Availability (single Region, keine Availability-Zone) => ~4 Minuten 20 Sekunden Downtime pro Monat. Zusätzlich können die Daten zu mehreren Azure-Regions repliziert werden.

**Speech Servies, Google-Authentifizierung & OpenAi**:
Sind alles externe Dienste, die eigene Maßnahmen zur Ausfallsicherheit bereits implementieren. Theoretisch könnte auch hier auf mehrere Azure-Regions/Endpunkte zurückgegriffen werden.

### NoSql. Welchen Beitrag leistet NoSql in der vorliegenden Problemstellung?

**Cosmos DB:**
Grundsätzlich hätte das Datenmodell auch mit einer relationalen Datenbank abgebildet werden können.
Allerdings lassen sich die Chats einfacher und kompakter als JSON-Dokument modellieren. Es werden dadurch Table-Joins vermieden und schnelle Queries gewährleistet.    
Zusätzlich haben wir die Flexibilität genutzt und verschiedene Typen in einem DB-Container gespeichert um Mehrkosten eines zusätzlichen Containers zu meiden. 

### Replikation. Wo nutzen Sie im gegenständlichen Projekt Daten-Replikation?

**Cosmos DB:**
Für den physischer Speicher vom Cosmos DB Container werden mind. 4 Replikationen garantiert und automatisch verwaltet. 

### Kosten. Welche Kosten verursacht Ihre Lösung? Welchen monetären Vorteil hat diese Lösung gegenüber einer Nicht-Cloud-Lösung?

**Theoretische Annahme:**
* 1000 (Power-)User pro Tag,
* Jeder User erstellt täglich:
  * 5 Chats mit je
  * 20 Interaktionen (Sprechen + Antwort vom Service)
  * mit einer Input-Sprechlänge von je 10 Sekunden + 20 Sekunden Antwort
  * ~1 Sekunde = 5 Zeichen Text
* Geschätzter Speicherverbrauch pro Operation (Eintrag einer Interaktion/eines Chats, **sehr größzügig**): 5 KB
* 1 Jahr Betrieb

**Azure App Services (Region 'West Europe'):**  
Abhängig von der Auslastung könnte zwischen den Basis- bzw. Premium-Plänen entschieden werden.
* `1000 * 20 * 5 = 100.000` Requests pro Tag
* Als Beispiel nehmen wir B2 für ein wenig mehr RAM und 2 Cores.

Geschätzte Kosten App-Services: 109.50$/Monat (B2)

![Kosten App Service](https://github.com/seventinnine/patter-pal/assets/18032233/63ffa667-5e58-48ef-aa9e-d04a7d7653e8)

**Cosmos DB (Region 'West Europe'):**
* Beispielsrechnung mit https://cosmos.azure.com/capacitycalculator
  * Throughput:
    * Free-Tier: bis 1000 RU/s
    * Request Unit: Normaisierung vom Aufwand von DB Operation, unterschiedliche Anzahl von RU werden verbraucht (https://learn.microsoft.com/en-us/azure/cosmos-db/request-units)
  * Annahme:
    * `1000 * 5 * 20 Creates/Tag` => ~1.2 Creates/Sekunde,
    * Jedes Create verursacht ~5 Point Reads (1 Item per ID nach dem erstellen wieder raus lesen, see https://devblogs.microsoft.com/cosmosdb/point-reads-versus-queries/ ) => ~6 Point Reads/Sekunde
    * Annahme: jeder User setzt 10 Queries pro Chat ab (z.b. Page reload) => `1000 * 5 Queries/Tag` => ~0.2 Queries/Sekunde
    * ~0.1 Updates/Sekunde (Renamen von Chats),
    * ~0.2 Deletes/Sekunde (Löschen eines Chats, Löschen aller Daten)
    * 20% Peak time, mit x5 so viel Requests
  * Storage:
    * Free Tier: bis 25 GB
    * Annahme:
     * `1.2 Creates/Sekunde * 3600 * 24` => `103'680 Creates/Tag * 31` => `3'214'080 Creates/Monat * 12` => `38'568'960 Creates/Jahr * 5 KB` => `192'844'800 KB` => ~183 GB => Aufrunden auf 200 GB für 1 Jahr Betrieb

Geschätze Kosten DB: 73.36$/Monat

![Kosten Cosmos DB](https://github.com/seventinnine/patter-pal/assets/58472456/ade47b5b-c41f-4aec-8654-68683b8bcfac)

**Azure Speech Services:**
* `100.000 / 2` Sprach-Input & Antworten 
* Speech to Text
  * `1.600 + 480 (extra Features Pronounciation etc.) = 2.080$/2.000 Stunden`
  * `50.000 * 10 = 500.000 Sekunden-Input/Tag / 60 / 60 = 138 Stunden/Tag ~= 4.000 Stunden/Monat`
  * `2 * 2.080 = 4.160$ Monat`
* Text to Speech
  * 1.024$ per 80 Millionen Zeichen
  * `50.000 * 20 * 5 = 5.000.000 Zeichen/Tag ~= 150.000.000 Zeichen/Monat`
  * `150 / 80 * 1.024 ~= 1.900$ Monat`

Geschätzte Kosten Speech Services: 6.060$/Monat.

![Kosten STT](https://github.com/seventinnine/patter-pal/assets/18032233/b9092969-6038-41bd-b554-ca6c4a468084)

![Kosten TTS](https://github.com/seventinnine/patter-pal/assets/18032233/81b84eb6-c249-4227-9058-080b92061b50)

**OpenAI**
* Annahme 1 Token ~5 Zeichen (Unterschätzung, damit es gleich ist mit Zeichen/Sekunde)
* Input
  * `50.000 * 10 = 500.000 Tokens/Tag Input`
  * `0.001$ pro 1.000 Tokens Input`
  * `0.001 * 500 = 0,5$ Tag ~= 15$/Monat`
* Output
  * `50.000 * 20 = 1.000.000 Tokens/Tag Output`
  * `0.002$ pro 1.000 Tokens Output`
  * `0.002 * 1.000 = 2$ Tag ~= 60$/Monat`
 
Geschätzte Kosten OpenAI: 85$/Monat.

![Kosten OpenAi](https://github.com/seventinnine/patter-pal/assets/18032233/5f41f71c-7be3-46bc-83d5-c5ede45e59d9)

**Gesamtkosten im Monat (grob): ca. 6.400$**  
So würde ein Abo für diesen Dienst für ein Break-Even rund *6,5$/Monat* kosten.

**Vorteil gegenüber Nicht-Cloud-Lösung:**
* Keine Hardware-Kosten notwendig für das Hosting der Services
* Kein selbst Trainieren/Hosten der Speech-Services und Large-Language-Models
* Einfache Skalierbarkeit (Scale out & up) ohne zusätzliche Hardware zu erwerben und warten zu müssen
* Schnelles Aufsetzen/Löschen von Ressourcen
* Ausfallsicherheit durch SLA

## Project Team
We are based in Austria and currently studying Software Engineering at the [University of Applied Sciences Upper Austria](https://www.fh-ooe.at/en/hagenberg-campus/).

**Members:**
- [Marcel Salvenmoser](https://github.com/malthee)
- [Stefan Weißensteiner](https://github.com/seventinnine)

## Testing Instructions

Visit [https://patter-pal.azurewebsites.net](https://patter-pal.azurewebsites.net)

Login:
- either use the Special Access Code we have provided or
- or connect your Gmail-Account with the application
  - PatterPal only requests the scope necessary for reading the email address from the token. PatterPal does not request any private user data

<img src="https://github.com/malthee/patter-pal/assets/58472456/72e03155-2a4a-4627-830f-2afcb681846a" alt="PatterPal Login"/>

Select a language of your choice and click the round button that resembles a microphone.

<img src="https://github.com/malthee/patter-pal/assets/58472456/ea3a090a-c29c-4bb5-a946-46a83ffce1d2" alt="PatterPal Language Select"/>

If this is your first visit (or depending on your browser settings), you will need to allow the website to use your microphone.

![Microphone Permissions](https://github.com/malthee/patter-pal/assets/58472456/5118ca73-dc6a-4de2-a55d-db6a7a2f96c5)

Click the 🎙️ button and start talking. A few seconds after speaking, your spoken text will gradually show up.
The recording will stop after some moments of silence or if you manually click the 🎙️ button again.

<img src="https://github.com/malthee/patter-pal/assets/58472456/6f671f3d-bf90-429a-9bbb-376b34b7070d" alt="PatterPal Start Recording"/>

A few seconds after the recording has halted, your language teacher will gradually respond.
After the response has finished generating, the response will be read to you via Speech-to-Text.
Below the language selelection box, you can the metrics regarding your spoken words. Misprounciations will also be highlighed in your spoken text.
If you want to stop the Speech-to-Text output, you can click the ✋ button.
Also keep in mind that you can change the langauge of the conversation whenever you want.

<img src="https://github.com/malthee/patter-pal/assets/58472456/97092df2-adba-4e39-b335-5472bce52c80" alt="PatterPal Responding"/>

If you want to your conversation history, you can press the 📃 button on the top right of the screen.
It toggles the your conversation history and allows you to start a new conversation
You can also rename or delete individual conversations here.

<img src="https://github.com/malthee/patter-pal/assets/58472456/1eb30964-cd7c-43e4-8fc5-cc2fd9060c83" alt="PatterPal Conversation History"/>

After you had a few conversations with your language teacher, you can visit the stats page by clicking the 📊 button on the top right.
Here you can see how your accuracy has changed over time or what words were least accurately pronounced.
You can filter your metrics by *language* and also adjust the analysed time period and time resolution (playing around with these values is a good idea if you've been PatterPal for an extended period of time).

<img src="https://github.com/malthee/patter-pal/assets/58472456/d366a003-8851-492f-bbc7-8ba0d801066c" alt="PatterPal Stats"/>

You can get back to the application by clicking the PatterPal icon on the top left.
When you are done you can use the 🚪 button on the top right to log out.

## Diagrams
### Azure Tech Stack
<img src="https://github.com/malthee/patter-pal/assets/18032233/04613ab0-afda-4120-95c1-308b205cbc79" alt="Azure Tech Stack"/>

*The OpenAi interface is not yet used from Azure (instead platform.openai.com) as it is not currently available for a students' subscription*

### Data Layer Diagram

![Data Layer Diagram](docs/img/datalayer.svg)

### WebSocket Communication Workflow
![WebSocket Communication](docs/img/communication.svg)

## Privacy Policy

### Information Collection
- Email-Based Accounts: If you register for an account using your email, we collect and store your email address. This is used for account verification.
- Conversations and Chats: We record and store the details of your conversations and chats with PatterPal. This includes both your input and the responses from PatterPal. This is required so you can access past conversations anytime.
- Pronunciation Analysis: When speaking with PatterPal, we collect and analyze data on your pronunciation accuracy and the mistakes made during spoken text exercises. This information is used to provide personalized feedback and improve your learning experience.

### How We Use Your Information
The information we collect is used to:
- Provide, operate, and maintain our services.
- Improve, personalize, and expand our services.
- Understand and analyze how you use our services.
- Develop new products, services, features, and functionality.
- Communicate with you for service-related purposes.

You can delete your *Conversation, Chat and Pronunciation Analysis* data on our [Privacy Page](https://patter-pal.azurewebsites.net/Home/Privacy).
