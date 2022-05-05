---
title: "DWE Sammel-App"
date: 2022-03-15T16:42:34+02:00
draft: true
author: "Simon"
weight: 5
---

_This article was originally written as a review of the DWE Sammel-App, which is the precursor to the Berliner Sammelapp. It's about the experiences of that developer team, which is still active with the Berliner Sammelapp to differing extents._

# Intro
## What's the DWE?
The campaign _Deutsche Wohnen & Co enteignen_ aims to communalize all big for-profit housing companies in Berlin in accordance with paragraph 14 of the German _Grundgesetz_. The relevant referendum was won on September 26 with 59% yes to 39% no.

## History of the app
To hold a referendum, a campaign had to first collect about 180k physical signatures. This campaign was organized via decentral neighbourhood teams. To support those teams, we developed the Sammel-App, which makes it easy for supportes to join signature collection drives.

### Who developed it?
Our team consisted of 3 people for most of the time, in total 5 people were involved. All of us have years worth of software development experience, though very little of that with app development.

### Why was it developed?
The original goal of the app was to be especially appealing to those supporters who only want to contribute every once in a while and do not have the time to get up-to-date information from the neighborhood team chats or the plenaries. The central usecase was to pick your neighborhood and receive a notification whenever somebody's doing a collection nearby. Over time, other features were added that were useful to the campaign.

### What are the features?
In the app, a user can receive information about the campaign, see planned actions, register their participation, chat with other participants, and announce actions themselves. It's also possible to register door-to-door conversations and the postering and removal of placards. A more complete feature overview is available in the [Play Store](https://play.google.com/store/apps/details?id=de.kybernetik.sammel\_app).

## Point of this article
We hope, that this article is interesting to various groups: Developers who are interested in Flutter can learn from our mistakes. Data analysts can see, what an app like this can and can't do for a campaign and what the user numbers of a successful campaign roughly look like. Activist developers can use this projects to reflect, whether technical work for a political campaign is worth the effort. People who organize a signature campaign themselves can assess, whether the Sammel-App might be useful for their project too.

# Technical Set-Up
## Structure
The app is a Flutter app. Every time the app is installed, a new, random user id is generated. This id is used for accessing the server, which is written in Kotlin.

## Hosting
The server is hosted via digitalocean.com for €10/month (€5/month didn't quite cut it). The server runs via Wildfly.

## Stores
Using the Flutter tools, we built both an iOS and an Android app and made them available on the Apple App Store and the Google Play Store as well as the Aurora Store. We didn't have experience with any of these. The Play Store was the easiest but still far more effort than we'd gleaned from reading the documentation.

The App Store was a nightmare. We learned, that there is no way of building an iOS app without a Macbook. We also realized that it's near impossible to have more than one person taking care of pushing new versions to the app store because distributing certificates becomes too hard. The App Store expects you to build an app with general appeal that you want to advertise and will beuseful to many different iPhone users. This is obviously not the case for the Sammel-App because our potential users are extremely limited from the getgo. You still have to make the effort to mat. screenshots, write advertising copy and fill in a pile of forms.

## Fehler
# Mistakes
Three times we caused an outage because we didn't replace the certificates on the server in time. Let's Encrypt certificates have such short expiry times specifically to force people like us to autmate the replacement process and we should've just done that. Luckily, switching certificates on the server is fast and easy.

Once the root certificate used in the app for verifying the server certificate expired. When setting up, whe should have noticted that we picked the one expiring in a year rather than a decade: [https://letsencrypt.org/docs/dst-root-ca-x3-expiration-september-2021/](https://letsencrypt.org/docs/dst-root-ca-x3-expiration-september-2021/)

For a while, our server kept crashing because it ran out of memory. Because we used a Java VM inside a Docker container, this was quite hard to debug. In the end, we solved it by upgrading the server. This lecture was helpful: [https://vimeo.com/181900266](https://vimeo.com/181900266)

For the door-to-door chats functionality we had to use Overpass by OSM, to turn click on a map into addresses. All publicly available servers were too slow, so we had to set up our own. When looking at other political apps, we saw that they usually don't bother with addresses and just place dots on the map - maybe that's the better approach.

When at one point we had to quickly push an update to the app, we failed ay doing this for the Play Store, since the only person with the magic upload key on their laptop was currently on vacation far away from it. It would be good for multiple people to have actually gone through the upload process (although Apple of course makes this intentionally hard.)

We were kicked from the Google Play Store one day without warning because Goople believed the app was trying to access background location without us having gained approval to do that. The app itself of course doesn't, but apparently one of the dependencies was not separated well and was flagged because of this. The incident was surprising to us because we hadn't expected that we could be kicked for a permission we neither requested nor wanted, with no time for review.

This incident came at a bad time right before a weekend camp of collecting signatures which was supposed to heavily rely on the app. Luckily, we could resolve the problem by distributing the APK file via messengers for sideloading.

At one point, we added a OWASP plugin to our server which automatically scans for dependencies with known security problems. Afterwards, our server could no longer properly serialize JSON. This probably has nothing to do with the plugin but with a combination of dependencies that get upgraded to a higher version, but for a long time, we did not know of any way to fix it besides removing the plugin. We still don't know, what exactly was the problem.

At the beginning of development, we took a very liberal attitude to Material Design (the design language recommended by Flutter). This worked well until later functionality ruined our UI.

It was quite tricky implementing the fairly complex navigation within the app, which includes redirects, in the very simple Flutter Navigator. In the end, we gave up and built our own navigation manager.

# Data
The app does not collect personally identifiable data. Users are assigned a random id on first start-up and can optionally pick an arbitrary, non-unique display name. We can still read quite a bit about usage from the logs and the store analytics.

## User development
In total about 4k people have installed the Sammel-App: about two thirds on iOS, one third on Android.

![Apple App Store installations per day, shows 5-20 each day, spikes of 50-100 in the last week of February and on April 15, smaller spike beginning of August, fade-out from July onwards](/images/apple_installs_per_day.png "Apple App Store installations per day, blue line")

![Google Play Store Installationen pro Tag, shows 5-20 each day, spikes of 50-100 in the last week of February and on April 15, smaller spike beginning of August, fade-out from July onwards](/images/google_installs_per_day.png "Google Play Store installations per day")

(Displaying the cumulative number is not possible for iOS because uninstalls are not logged, so we chose to only look at additive installs for both.)

On start-up, the app authenticates to the server with the user id and the corresponding password saved in the app. After removing same-day repeat logins from this data, we are left with the daily active users:

![Daily active users, shows 200-600 per day with spikes on weekends, downtime between mid-June and August, almost no usage after October](/images/DAUs.png "daily active users")

We see that the app was used by about 200 people every day. On some weekends the usage reaches up to 600 people.

Spikes in usage can be explained with the occasional global push notifications, which are addressed to all users and will often lead them to open the app.

In both data sets, we can see a spike on April 15 – the day the Federal Constitutional Court declared the Berlin rent cap as unconstitutional. About 200 people decided to install the app on this day.

From the daily usage numbers, we can also see that about 1k users opened the app on at least 10 different days. We could consider these regular users.

## Survey
In the middle of the campaign, we asked users to participate in a survey to help us better understand app usage. About 40 people participated, of which 70% indicated using the app at least once a week. 30% said they are not active in the campaign itself and only participate via the app.

## Collection efficency
The app would ask people after participating in a signature collection drive to evaluate the event and say how many signatures they collected. This survey was optional and users would frequently make guesses – many people just hand in their lists at some point and don't even know, how long they were collecting or how many signatures their group collected. Nevertheless we can provide some orders of magnitude:

![Efficiency by district, shows the average of collected signatures per person-hour, between 10 and 30 in all districts](/images/sammeleffizienz_nach_bezirken.png "Average collected signatures per person-hour by district")

Our interpretations is:
   * One person can expect to collect between 10 and 25 signatures in an hour.
   * It can make a difference, in what district one goes collecting, but it's not large.

We also looked at efficiency over time. It did not vary much.

It also seemed like there's no large difference in efficiency depending on group size. If anything, it seems like efficiency decreases a bit for groups larger than 10.

# Die App & DWE
# Sammel-App & DWE
## How useful was the app for the campaign as a whole?
From the numbers above we can see that the app was used frequently and was at least for some the primary way of interacting with the campaign. We see three forms of impact: reach, branding, efficiency.

We can see that the app reached some people who would not otherwise have participated in the campaign. We also believe that some people participated in more actions than they would have without the app, simply because the reached timely information about contribution possibilities.

From talking to activists, we have learned that the mere existence of an app with decent design conveys a sense of professionalism – some activist was suprised, "that the DWE campaign even has an app.". We believe that the app cotributed to an overall perception of a well-organized campaign and thereby motivated people to participate who otherwise would not enjoy decentralized left-wing grassroots work.

Lastly we saw that the app was used for some very particular pain points and small projects. This includes a weekend of focused collection work and the placard functionality, which made coordinating the postering and removal of placards easier.

## Is is worth doing data analysis for a campaign like this?
One goal at the start of development was to be able to get better data through the app about the efficency of signature collection actions, in particulary with respect to time and place. Looking back, we'd say that the data from the app is of limited use for this. The analysis above is the best we could do. In partictual any statement about what intersection or times of day are particularly good is not possible from the app data. The data itself is simply too bad – voluntary self-assessments about for how long somebody was working or how many signatures they collected simply are not that reliable.

It might be possible to improve the data by not only asking for voluntary self-evaluations, but making the app itself the central reporting tool for the campaign. At the DWE campaign, the reporting was done manually, with each district team one a week collecting alll signatures that somehow made it to them by that time.

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
