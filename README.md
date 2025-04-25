# OptimusSecurity

### Kurzfassung

OptimusSecurity ist ein bahnbrechendes OIDC-Provider-Tool, das eine vollständig automatisierte, mehrstufige Sicherheitslösung bereitstellt. Der Registrierungsprozess erfolgt papierlos: Benutzer laden eine Word-Vorlage von der offiziellen Website herunter, füllen diese elektronisch aus und laden sie wieder hoch. Ein intuitives Verwaltungstool zeigt eingehende Anfragen an, die nach individueller Prüfung freigegeben werden. Bei Freigabe wird im Hintergrund automatisch ein digitales Zertifikat samt verschlüsselter Authentifizierungsschlüssel generiert – wobei sowohl JWT-Tokens als auch Refresh-JWT-Tokens verwendet werden, um sichere und kontinuierliche Sessions zu gewährleisten.

Besonders innovativ ist der Einsatz eines speziell konfigurierten USB-Sticks: Er wird verschlüsselt ausgeliefert, und das zugehörige, zusätzlich verschlüsselte Schlüsselmaterial – das ausschließlich in Kombination mit dem OIDC-Provider entschlüsselt werden kann – wird nur dem registrierten Benutzer zur Verfügung gestellt. Das Stick-Passwort wird per SMS versendet, und der Schlüssel wird unter Einbindung benutzerspezifischer Daten sowie der Hardware-Nummer des angemeldeten Laptops generiert, sodass sichergestellt ist, dass nur dieses Gerät verwendet werden kann. Das Einmalpasswort erfolgt per E-Mail als zusätzlicher Faktor – nutzbar ausschließlich, wenn der USB-Stick eingesteckt ist – während Browser-Cookies streng mit HttpOnly, Secure und SameSite konfiguriert sind.

Die Projektarchitektur umfasst vier Angular-Webseiten (Angular 18 non-module), vier Microservice-Projekte (SpringBoot Java 21) mit RabbitMQ, ein API-Gateway, zehn API-Projekte (ebenfalls SpringBoot Java 21), sowie einen dedizierten oidc-provider und eine oidc-client-api. Ergänzt wird dies durch fünf Batch-Anwendungen (SpringBatch Java 21) und ein umfangreiches Testkonzept, das Unit-Tests, JUnit-Tests, Seleniumtests (Penetration, GUI, Bots), DataQuality-Tests und JMeter-Last- und Performancetests einschließt. Zusätzlich verwendet das Projekt moderne Tools wie local-nexus, Git/GitHub, Docker und Docker-Compose sowie eine H2-Datenbank. Unterstützt wird die Lösung durch umfassende Dokumentation in Form von Konzepten, Benutzerhandbüchern, UML-Diagrammen und Einführungsvideos.

Mit seiner modularen, hochsicheren und benutzerzentrierten Architektur bietet OptimusSecurity eine zukunftssichere Lösung, die höchste Sicherheitsstandards mit einer entwicklerfreundlichen Implementierung verbindet – ein echter Mehrwert für jedes moderne Unternehmen.

### Projektbeschreibung

OptimusSecurity ist ein OIDC-Provider-Tool, das die gesamte Sicherheitsverwaltung für den Benutzer übernimmt. Der Benutzer ruft die offizielle Website https://www.optimussecurity.ch/ auf, lädt dort ein Word-Dokument für Benutzeranfragen herunter, füllt dieses elektronisch aus und lädt es anschließend über die Website wieder hoch.

Mitarbeiter von Novaproduction/OptimusSecurity arbeiten mit einem Verwaltungstool, in dem die eingegangenen Benutzeranfragen tabellarisch dargestellt werden. Anschließend nimmt Novaproduction Kontakt mit dem Unternehmen des Benutzers auf, um alle Einzelheiten zu besprechen. Sobald alle Punkte geklärt sind, wird der neue Benutzer im Verwaltungstool freigegeben. Im Hintergrund wird dabei automatisch ein neues Zertifikat samt Authentifizierungsschlüsseln generiert und zusammen mit den Benutzerdaten im System abgespeichert.

Besonders hervorzuheben ist, dass der USB-Stick, auf dem der Public Key verschlüsselt gespeichert wurde, zusätzlich verschlüsselt ausgeliefert wird. Das zugehörige Passwort wird dem Benutzer per SMS zugesendet, sodass der USB-Stick ausschließlich dem vorgesehenen Empfänger zugeordnet ist. Der auf dem Stick gespeicherte Schlüssel wird nochmals verschlüsselt und kann nur in Kombination mit dem OIDC-Provider verwendet werden, der als einziger in der Lage ist, diesen Schlüssel zu entschlüsseln. Zudem wird der Schlüssel unter Einbeziehung personenbezogener Informationen generiert, sodass allein der jeweilige Benutzer ihn nutzen kann. Darüber hinaus wird eine Hardware-Nummer des Laptops des Benutzers hinterlegt, wodurch sichergestellt wird, dass ausschließlich das registrierte Arbeitsgerät verwendet werden kann.

Zusätzlich erhält der Benutzer eine E-Mail mit einem einmaligen Passwort, das als zweite Authentifizierungsebene dient. Dieses Einmalpasswort ist jedoch nur nutzbar, wenn der Benutzer den verschlüsselten USB-Stick eingesteckt hat und sich an seinem Registrierungs-Laptop anmeldet – so fungiert die E-Mail auch als 2-Faktor-Authentifizierung.

Der Entwickler muss lediglich die optimus-oidc-api in seine eigene Website integrieren – diese kann bequem über Maven bezogen werden. Sämtliche nötige Konfigurationen werden über eine Property-Datei vorgenommen, sodass sich der Entwickler nicht um die Sicherheit, Token-Erstellung oder Definition von Endpoints kümmern muss.

Möchte sich der Benutzer auf https://www.clientpage.ch/ anmelden, wird er automatisch auf die Login-Seite weitergeleitet. Dabei ist es zwingend erforderlich, dass der per Post zugesandte USB-Stick eingesteckt ist (nur dieser USB-Stick ist zugelassen). Der Benutzer gibt seine E-Mail-Adresse als Benutzernamen sowie das in der E-Mail erhaltene Einmalpasswort ein und klickt auf „Login“. Im Backend erfolgt dann die Verifizierung der Anfrage beim OIDC-Provider. Bei erfolgreicher Validierung wird ein Verifizierungscode an die E-Mail-Adresse des Benutzers versendet und gleichzeitig die Code-Verifizierungsseite geöffnet, auf der der Benutzer den erhaltenen Code eingeben muss.

Ist der Verifizierungscode korrekt, wird der Benutzer – ausschließlich bei der Erstanmeldung – aufgefordert, sein Passwort neu zu setzen. Hierzu öffnet sich die Seite „Passwort-Reset“, auf der das neue Passwort zweimal eingegeben und bestätigt wird. Anschließend generiert das System einen Initialcode, der zusammen mit den anderen Benutzerdaten an einen definierten Endpunkt der oidc-client-api übermittelt wird. Diese API speichert die Daten in Cookies, die unter strengen Sicherheitsvorgaben als HttpOnly, Secure und mit SameSite-Attributen konfiguriert sind. Parallel dazu liest die oidc-client-api die USB-Stick-Daten ein und speichert den Serial Number Code des USB-Sticks in der Benutzersession.

Sobald der Initialcode in den Cookies hinterlegt ist, kontaktiert die oidc-client-api den OIDC-Provider, um Initialtokens anzufordern – dabei werden die Benutzerdaten (einschließlich der USB-Stick-Serialnummer) und der Initialcode übermittelt. Der OIDC-Provider generiert daraufhin einen Initialtoken sowie einen Refreshtoken (wobei letzterer ausschließlich zur Anforderung weiterer Initialtokens verwendet wird) und stellt diese über einen definierten Endpunkt bereit. Die Tokens werden an die oidc-client-api zurückgesendet und in den Browser-Cookies abgelegt.

Wenn der Initialtoken abläuft, sendet die oidc-client-api eine Anfrage an den OIDC-Provider, um mithilfe des Refreshtokens neue Tokens zu erhalten. Dieser wiederkehrende Prozess gewährleistet, dass sich der Benutzer nicht bei jeder neuen Sitzung manuell anmelden muss. Der Token kann maximal fünfmal regeneriert werden – danach verfällt er und wird aus dem Cookie gelöscht, sodass eine erneute Anmeldung erforderlich wird.

Nach erfolgreicher Registrierung beim OIDC-Provider wird der Benutzer automatisch auf https://www.clientpage.ch/ weitergeleitet. Die oidc-client-api prüft anhand der Cookies, ob der Benutzer bei OptimusSecurity angemeldet ist. Ist dies der Fall, wird der aktuell gültige Initialtoken auf seine Gültigkeit und seinen Ablauf überprüft. Anschließend erfolgt die Überprüfung, ob der Benutzer über die für den Zugriff erforderlichen Rollen verfügt, wie sie in seinem Zertifikat hinterlegt sind. Nur Benutzer mit den entsprechenden Berechtigungen erhalten Zugang zur Hauptseite und werden automatisch angemeldet. Da der gesamte Anmeldeprozess nahtlos im Hintergrund abläuft, erscheint auf https://www.clientpage.ch/ kein herkömmliches Login-Fenster – der Benutzer bemerkt den Wechsel zwischen den Seiten nicht.

Um Fehlalarme zu filtern, erhält der zuständige Mitarbeiter im Verwaltungstool eine detaillierte Übersicht über sämtliche Aktionen des Benutzers. Erkennt er, dass es sich nicht um eine Sicherheitsverletzung handelt, kann er den Vorfall im Tool abschließen. Zusätzlich werden Analysen und Reviews jede Nacht durch automatisierte Datenbank-Batches durchgeführt, die Sicherheitsprüfungen sowie die Überwachung der täglichen Abläufe und der Datenqualität vornehmen. Diese Maßnahmen stellen sicher, dass Unregelmäßigkeiten frühzeitig erkannt und die Datenqualität konstant überwacht wird.


### Das Projekt umfasst

```

4 Webseite (Angular 18 non-module)
4 Microservice Projekte (SpringBoot Java 21) - RabbitMQ
1 Api-Gateway (SpringBoot Java 21)
10 api Projekte (SpringBoot Java 21)
1 oidc-provider
1 oidc-client-api
5 Batche (SpringBatch Java 21)
Unit-tests
JUnit-tests
Seleniumtests (Penetrationtests, GUI-Tests, Bots)
DataQualityTests
Jmeter Last&Performancetests
local-nexus
git
GitHub
docker
docker-compose
h2 database

```
