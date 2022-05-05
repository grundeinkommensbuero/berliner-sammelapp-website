---
title: "DWE Sammel-App"
date: 2022-03-15T16:42:34+02:00
draft: true
author: "Simon"
weight: 5
---
_Dieser Artikel wurde ursprünglich als Revue der DWE Sammel-App geschrieben, die der Vorgänger der Berliner Sammelapp ist. Er erzählt die Erfahrungen des damaligen Entwicklerteams, das auch bei der Berliner Sammelapp noch in verschiedenem Maße aktiv ist._

# Intro
## Was ist die DWE?
Die Kampagne _Deutsche Wohnen & Co enteignen_ zielt auf die Vergesellschaftung nach Paragraph 14 des Grundgesetzes aller großen, profitorientierten Wohnungskonzerne in Berlin ab. Ein entsprechender Volksentscheid wurde am 26. September mit 56% zu 39% gewonnen.

## Geschichte der App
Vor dem Volksentscheid musste eine Unterschriftenkampagne zunächst 180k Unterschriften sammeln. Diese Kampagne wurde organisiert über dezentrale Kiezteams. Zu ihrer Unterstützung haben wir die Sammel-App entwickelt, über die interessierte Menschen einfach an Sammel-Aktionen teilnehmen konnten.

### Wer hat sie entwickelt?
Unser Team bestand die meiste Zeit aus drei Menschen, insgesamt haben 5 daran mitgearbeitet. Wir haben alle mehrjährige Erfahrung mit Softwareentwicklung, allerdings nur sehr wenig davon mit App-Entwicklung.

### Warum wurde sie entwickelt?
Ursprüngliches Ziel der App war es, besonders für solche Menschen ansprechend zu sein, die nur hin und wieder einmal aktiv sein wollten und nicht die Zeit hatten, sich in den Kiezteamchats und auf den Plena die aktuellen Infos zu holen. Der zentrale Usecase sollte sein, dass man seinen Kiez eintragen kann, und immer eine Notiz bekommt, wenn dort eine Sammel-Aktion stattfindet. Über die Zeit kamen dann weitere Features dazu, die für die Kampagne nützlich waren.

### Was kann sie?
Über die App kann man Informationen zum Volksbegehren erhalten, geplante Aktionen sehen, seine Teilnahme registrieren, mit anderen Teilnehmenen chatten und selbst Aktionen einstellen. Außerdem können Haustürgespräche registriert werden und das Auf- und Abhängen von Plakaten koordiniert. Eine Übersicht kann man im Play Store Eintrag sehen: [https://play.google.com/store/apps/details?id=de.kybernetik.sammel\_app](https://play.google.com/store/apps/details?id=de.kybernetik.sammel\_app)

## Sinn dieses Beitrags
Wir hoffen, dass dieser Artikel für verschiedene Gruppen interessant ist. Entwickler:innen, die an Flutter interessiert sind, können hoffentlich von unseren Fehlern lernen. An Zahlen interessierte Menschen können nachvollziehen, was so eine App für eine Bewegung leisten kann und nicht leisten kann, und wie die Nutzungszahlen bei einer erfolgreichen Kampagne ungefähr aussehen. Aktivistische Softwareentwickler:innen können an diesem Projekt reflektieren, ob sich technische Arbeit für einen politischen Zweck lohnt oder es eher nicht wert ist. Menschen, die selber Unterschriftenkampagnen organisieren, können sich überlegen, ob die Sammel-App vielleicht auch für ihr Projekt hilfreich sein könnte.

# Technischer Aufbau und Fehler
## Struktur
Die App ist als Flutter-App entwickelt. Bei jeder Neuinstallation wird eine neue, zufällige Nutzer-ID vergeben, die die Nutzerin identifiziert. Diese ID regelt den Zugriff auf den Server, der in Kotlin geschrieben ist.

## Hosting
Wir hosten unseren Server über digitalocean.com für €10/Monat (der Server für €5 Monat hat nicht ganz ausgereicht). Der Server läuft via Wildfly.

## Stores
Wir haben die App mit den Flutter-Tools in eine iOS-App und eine Android-App kompiliert und sowohl auf dem Google Playstore als auch auf dem Apple App Store verfügbar gemacht. Als Google-freien Store konnten wir den Aurora Store bewerben. Wir hatten mit nichts davon Erfahrung – der Playstore war bei weitem am Einfachsten und verständlichsten, aber immer noch wesentlich mehr Aufwand, als wir das aus der Dokumentation hätten erahnen können.

Der Apple Appstore war ein Alptraum. Wir mussten feststellen dass es keine Möglichkeit gibt eine iPhone-App ohne einen Mac zu bauen.  Wir haben gelernt, dass praktisch immer nur eine Person die Uploads in den Apple Appstore übernehmen kann, weil das Zertifikate verteilen ansonsten zu kompliziert wird. Der Appstore erwartet, dass man eine allgemein ansprechende App gebaut hat, die man bewerben will und auch für viele Apple-Nutzer einen Mehrwert bringt. Das ist bei der Sammel-App offensichtlich nicht der Fall, weil unser Nutzer:innenkreis von vornherein stark begrenzt ist. Trotzdem ist es großer Aufwand, die entsprechenden Screenshots zu machen, Werbetexte zu schreiben,  Formulare auszufüllen und Anleitungsvideos zu erstellen, die Apple für den Appstore erwartet.

## Fehler
Wir haben dreimal eine Outage gehabt, weil wir verraft haben, dass Let's Encrypt Zertifikat auf dem Server rechtzeitig auszuwechseln. Eigentlich haben LE-Zertifikate so kurze Laufzeiten, um zu erzwingen, dass man das automatisiert – das hätten wir auch einfach machen sollen. Zum Glück kann man die Zertifikate auf dem Server schnell wechseln.

Einmal ist uns das Root-Zertifikat in der App, mit dem die App das Server-Zertifikat verifiziert, ausgelaufen. Beim Aufsetzen hätte es uns vielleicht auffallen sollen, dass wir das mit einer Laufzeit von noch einem Jahr statt grob einem Jahrzehnt genommen haben. [https://letsencrypt.org/docs/dst-root-ca-x3-expiration-september-2021/](https://letsencrypt.org/docs/dst-root-ca-x3-expiration-september-2021/)

Unser Server ist uns eine Weile lang immer wieder gecrasht, weil der Arbeitsspeicher nicht ausgereicht hat. Weil wir eine Java VM hatten, die wiederum in einem Docker Container lief, war das super schwer zu debuggen. Wir haben das gelöst, indem wir den Arbeitsspeicher verdoppelt haben. Dieser Vortrag war dazu hilfreich: [https://vimeo.com/181900266](https://vimeo.com/181900266)

Für die Haustürgesprächsfunktionalität mussten wir Overpass von OSM nutzen, um aus Klicks auf der Karte Adressen zu machen. Alle öffentlich verfügbaren Server waren zu langsam, deswegen mussten wir schließlich unseren eigenen aufsetzen. Bei anderen politischen Apps haben wir gesehen, dass diese Auflösung gar nicht erst versucht wird, und man stattdessen einfach Punkte auf der Karte bekommt – vielleicht ist das der bessere Ansatz.

Als wir einmal sehr schnell wegen eines der obigen Probleme ein Update für die App pushen mussten, ist das für den Playstore daran gescheitert, dass zwar alle Zugriff hatten, aber der magische Upload-Key nur auf dem Laptop einer Person war, die gerade im Urlaub weit weg davon war. Es ist vermutlich gut, dass mehrere Leute mal einen Push-to-Production Prozess durchmachen.

Wir wurden aus dem Google Playstore gekickt, weil die App in Googles Augen eine Background-Location abfragte, ohne dass wir dafür ein Google-Formular ausgefüllt hätten. Die App selbst macht das jedoch gar nicht, allerdings war eine der Dependencies der App wohl nicht sauber genug abgegrenzt und wurde deswegen geflagged. Das war für uns überraschend, weil uns gar nicht klar war, dass wir für eine Permission, die wir gar nicht erfragt hatten oder nutzen wollten, aus dem Playstore fliegen konnten.

Dieses Problem traf uns zu einem kritischen Zeitpunkt, genau zum großen Sammelcamp, bei dem die Appelle zentrale Rolle spielen sollte. Wir konnten die Aktion dank der Sideloading-Freiheit von Android mit dem einfachen Verteilen einer APK-Datei über Messenger retten. 

Irgendwann haben wir ein OWASP-Plugin in unseren Server eingebaut, das automatisch nach Dependencies mit bekannten Sicherheitsproblemen scannt. Danach konnte unser Server kein JSON mehr serialisierem. Das hat vermutlich nichts mit dem OWASP-Plugin zu tun, sondern mit irgendeiner Verkettung von Dependencies, die dabei auf eine höhere Version gezogen wurde, aber wir haben lange keinen anderen Weg gefunden, als den Branch auf den vorherigen Stand zu zurückzusetzen. Wir wissen immer noch nicht, was genau das Problem war.

Wir haben uns am Anfang der App-Entwicklung relativ liberal zu Material Design (der von Flutter vorgesehen Designsprache) verhalten. Das hat auch gut funktioniert, bis uns spätere Funktionalität unsere UI versaut hat.

Ein großes Problem war die relativ komplexe Navigation der App, die etwa das Weiterleiten auf bestimmte Pages vorsieht mit dem sehr simplen nativen Futter Navigator zu verwirklichen. Schließlich haben wirst diesen weitestgehend verzichtet und unseren eigenen Navigationsmanager implementiert.  

# Daten
Die App erhebt und verarbeitet keine personenbezogenen Daten. Nutzer:innen bekommen beim ersten Start der App eine zufällige ID zugewiesen und können sich optional einen beliebigen, uneindeutigen Namen geben. Aus den Logs und den Zahlen der Stores können wir trotzdem einiges über die Nutzung der App ablesen.

## Nutzer:innenentwicklung
Insgesamt haben ca. 4k Menschen die Sammel-App installiert, davon ca. zwei Drittel auf iOS, der Rest auf Google Android.

![Apple App Store Installationen pro Tag, zeigt 5-20 jeden Tag, mit Spikes auf 50-100 in der letzten Februarwoche, am 15. April und kleinem Spike zum Anfang August, Abklingen ab Juli](/images/apple_installs_per_day.png "Apple App Store Installationen pro Tag, blaue Linie")

![Google Play Store Installationen pro Tag, zeigt 5-20 jeden Tag, mit Spikes auf 50-100 in der letzten Februarwoche, am 15. April und kleinem Spike zum Anfang August, Abklingen ab Juli](/images/google_installs_per_day.png "Google Play Store Installationen pro Tag")

Beim Öffnen authentifziert sich die App bei unserem Server mit der Nutzer-ID und dem in der App generierten und hinterlegten Passwort. Wenn wir aus diesen Logs Mehrfachlogins am selben Tag herausrechnen, bleibt uns die Zahl der täglich aktiven Nutzer:innen:

![Täglich aktive Nutzer:innen, zeigt zwischen 200-600 am Tag, mit Spikes am Wochenende, Niederaktivitäsphase zwischen Mitte Juni und Anfang August, fast keine Nutzung ab Oktober](/images/DAUs.png "Täglich Aktive Nutzer:innen")

Wir sehen, dass die App normalerweise von ungefähr 200 Menschen am Tag genutzt wurde. An manchen Wochenende ging die Nutzung auf ungefähr 600 Menschen hoch.

Spikes bei der Nutzung lassen sich mit den gelegentlichen globalen Push-Nachrichten erklären, die an alle Nutzer:innen adressiert sind und diese zum Öffnen der App animieren. 

Bei beiden Datensätzen erkennen wir einen Spike am 15. April – dem Tag der Entscheidung des Bundesgerichtshof, den Berliner Mietendeckel als verfassungswidrig einzustufen. Ungefähr 200 Menschen haben an diesem Tag beschlossen, auch die App zu installieren.

Über die täglichen Nutzungszahlen können wir auch auslesen, dass ungefähr 1k Nutzer:innen die App an mindestens 10 verschiedenen Tagen genutzt haben – wir würden sie entsprechend als regelmäßige Nutzer:innen einstufen.

## Umfrage
In der Mitte der Kampagne haben wir um die Teilnahme an einer Umfrage geben, um die Nutzung der App besser einschätzen zu können. 40 Menschen haben daran teilgenommen. 70% von ihnen haben angegeben, die App mindestens einmal pro Woche zu nutzen. 30% haben angegeben, dass sie nicht in der Kampagne selbst aktiv sind, sonder nur über die App.

## Sammeffizienz
In der App wurden Menschen nach Teilnahme an einer Sammelaktion darum gebeten, die Aktion zu evaluieren und anzugeben, wie viele Unterschriften gesammelt wurde. Diese Angabe war freiwillig und oft geschätzt – viele Menschen geben beim Sammeln einfach irgendwann ihre Listen ab und wissen gar nicht genau, wie lange sie am Sammeln waren und wie viele Unterschriften ihre Gruppe bekommen hat. Trotzdem können wir einige Größenordnungen angeben:

![Sammeleffizienz nach Bezirken, zeigt den Durchcshnitt der gesammelten Unterschriften pro Personenstunde, zwischen 10 und 30 in allen Bezirken ](/images/sammeleffizienz_nach_bezirken.png "Sammeleffizienz nach Bezirken")

Unsere Interpretation dazu ist:
   * Eine Person kann davon ausgehen, in einer Stunde zwischen 10 und 25 Unterschriften zu sammeln.
   * Es kann einen Unterschied geben, in welchem Bezirk man sammelt, aber der Effekt ist nicht groß.

Wir haben uns auch die Entwicklung der Sammeleffizienz über die Zeit angeschaut. Sie hat sich nicht stark verändert.

Außerdem können wir sehen, dass es keinen großen Unterschied bei der Effizienz gibt, ob in großen oder kleinen Gruppen gesammelt wird. Am ehesten wirkt es noch so, als ob ab 10 Leuten die Effizienz etwas abnimmt.

# Die App & DWE
## Wie hilfreich war die App für die Kampagne als Ganzes?

Aus den Zahlen oben können wir ablesen, dass die App rege genutzt wurde und zumindest für manche der primäre Bezug zur Kampagne waren. Wir sehen drei Wirkungswege: Reichweite, Branding, Effizienz.

Wenn wir den obigen Zahlen folgen, hat die App zumindest einige Menschen erreicht, die sich sonst nicht an der Kampagne beteiligt hätten. Wir gehen außerdem davon aus, dass manche Menschen an mehr Aktionen teilgenommen haben, als sie es sonst getan hätten, weil sie über die App zielgenau informiert wurden.

Aus Erzählungen haben wir gelernt, dass alleine die Existenz einer App mit einigermaßen ansprechender Gestaltung ein Gefühl von Professionalität vermittelt – manch ein Aktivist ist überrascht, dass „die DWE ja sogar eine App hat.“ Wir vermuten, dass die App zum Gesamtbild einer wohlorganisierten Kampagne beitragen hat und damit auch Menschen motiviert hat, die mit weniger organisierter linker Basisarbeit nicht so viel anfangen können.

Zuletzt haben wir erlebt, dass die App für einige sehr konkrete Projekte genutzt wurde und sie einfacher gemacht hat. Dazu zählt zum Beispiel ein Sammelaktionswochenende, bei dem die App der zentrale Infopunkt war, aber auch die Plakatfunktionalität, über die das Auf- und Abhängen koordiniert werden konnte.

## Macht es eigentlich Sinn, für so eine Kampagne Datenauswertung zu machen?
Ein Ziel zu Beginn der Entwicklung war, über die App bessere Daten über Sammeleffizienz erheben zu können, insbesondere was Zeit und Ort der Sammelaktionen angeht. Hier würden wir sagen, dass die Daten aus der App nur sehr eingeschränkt nutzbar sind. Die Datenanalyse oben ist im Grunde das Beste, was wir sagen können. Insbesondere Einschätzungen dazu, an welchen Kreuzungen oder Tageszeiten man am besten sammelt, sind mit den Daten aus der App nicht möglich. Hierfür ist die Datenlage einfach zu schlecht – freiwillige Selbsteinschätzungen dazu, wie lange oder wie viel jemand gesammelt hat, sind einfach nicht besonders zuverlässig.

Unter Umständen könnte man die Datenlage verbessern, wenn die App nicht nur freiwillige Evaluationen erfassen würde, sondern selbst das zentrale Werkzeug für das Reporting über den Status der Kampagne wäre. Bei der DWE wurde das manuell gemacht, das heißt, es wurden einmal pro Woche alle Unterschriftenlisten zusammen ausgezählt, die bis dahin irgendwie beim jeweiligen Kiezteam gelandet waren. 

## Was hätte besser laufen können?
Abgesehen von den oben benannten technischen Sorgen gab es noch ein paar andere Dinge, die wir rückblickend anders gestaltet hätten. Am wichtigsten ist vor allem, dass, wenn wir am Anfang gewusst hätten, wie gut die App funktionieren würde, wir ihr ein stärkeres Gewicht gegenüber anderen Kommunikations- und Organisationskanälen gegeben hätten. Das heißt einerseits, dass mehr Kiezteams ihre Aktionskoordination primär über die App gestalten hätten, und andererseits, dass wir vielleicht auch manches Reporting schon direkt über die App hätten machen können.

Davon abgesehen wäre eine stärkere Integration mit den anderen Kanälen sinnvoll gewesen. Das sind insbesondere Telegram-Kanäle und Etherpads gewesen. Wir haben erst relativ spät Sharelinks für Aktionen erstellt, das hätte die Verbindung zu Pads einfacher machen können. Außerdem wäre es sinnvoll gewesen, zumindest eine rudimentäre Browserschnittstelle für die Appdaten zu haben, so dass auch jemand ohne ein Telefon über den Laptop mit einem Link auf die Daten einer Sammelaktion sehen kann.

# Allgemeine Betrachtungen zu Softwareaktivismus
## Warum gibt es so viele Versionen dieser App?
Angesichts der Tatsache, dass nahezu jede Kampagne und jede Partei, die an Haustüren klopft, die gleiche Funktionalität braucht, hätten wir erwartet, dass es eine freie Standardlösung gibt. Stattdessen rollt jede Partei ihre eigene Version aus. Wir vermuten, dass es mehrere Gründe hierfür gibt: Einerseits ist es immer interessanter, selbst etwas zu bauen, als sich in fremden Code einzulesen. Das gilt für kostenlos arbeitende Entwickler:innen genauso wie für Parteien, denen von Softwareanbietern etwas vorgeschlagen wird. Außerdem ist die Integration mit bestehender Infrastruktur, beispielsweise Mitgliederverwaltungssystem auf Kreisverbandebene, ein großer Teil der Arbeit. Zuletzt ist das tatsächlich Featureset der App tatsächlich sehr spezifisch (Plakate, Haustürgespräche, Evaluationen) und eine Anpassung einer bestehenden Lösung dauert möglicherweise genauso lange wie ein Neubau.

## Ist es sinnvoll für aktivistische Softwareentwickler:innen, so eine App zu bauen?
Wir sehen uns als politische Aktivisten und waren alle auch in anderen politischen Kontexten auf andere Art und Weise bereits aktiv. Ein Gedanke hinter dieser App war auch, dass es eine effektivere Nutzung unserer Zeit ist, sie zu bauen, als uns anderweitig zu engagieren. Wir schätzen, dass wir zusammen ungefähr 1000 Stunden an der App gearbeitet haben. Hätten wir die Zeit mit dem Sammeln von Unterschriften verbracht, wären das etwas 15k zusätzliche Unterschriften gewesen, also etwa 5% mehr, als die DWE tatsächlich erreicht hat. Hätten wir stattdessen unsere Stunden in der bezahlten Arbeit aufgestockt, wäre bei bei einem Netto von €30/Stunde etwa €30k an zusätzlichen Spenden für die DWE herausgekommen – etwa das dreifache, was die Linke in Berlin an die DWE gespendet hat.

## Wie interagiert man als Techteam am Besten mit anderen aktivistischen Gruppen?
Unser Wunsch wäre, dass die App auch für andere sinnvolle Volksbegehren angepasst und verwendet wird. Die Idealvorstellung ist dabei natürlich, dass eine technisch versierte Person aus der Kampagne sich eigenständig um das Anpassen des Brandings, das Hosting, und das Konfigurieren der Store-Einträge kümmert. In unserer bisherigen Erfahrung klappt das eher mau. Die Kampagnen sind normalerweise sehr gestresst und das Konfigurieren der App, selbst wenn wir anbieten, uns für sie darum zu kümmern, nimmt keine Priorität ein.
