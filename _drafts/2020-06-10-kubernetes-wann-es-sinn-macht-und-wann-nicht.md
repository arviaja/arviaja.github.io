---
layout: post
title:  "Kubernetes - wann es Sinn macht und wann nicht"
date:   2020-06-10 12:36:00 +0200
categories: kubernetes docker paas sdlc continuous deployment
---
Docker und Kubernetes sind Schlagwörter, welche in den letzten fünf Jahren auf ein stetig steigendes Interesse stoßen (siehe dazu Abb. 1).

../assets/img/2020-06-10-google-trends-kube-docker.png
(Abb. 1)

Es erfährt zum Glück nicht den Hype (und dann den Absturz) wie viele andere IT-Trends der letzten Jahre, aber das Interesse wird größer und konkreter. Das sehe ich alleine in den deutlich gestiegenen Anfragen und Aufträge zu diesem Thema.

Kubernetes verspricht eine hochskalierbare, hochresiliente Entwicklungs- und Serviceplattform (Platform as a Service, PaaS), die, wenn einmal richtig eingespielt, einen Software Development Life Cycle (SDLC) erlaubt der ohne die gefürchteten Downtimes, Frozen Zones und Wartungsfenster auskommt und quasi unzerstörbar ist.

Aber wo Licht ist, ist auch Schatten, das vorweg. Kubernetes ist komplex und benötigt eine etwas andere IT Organisation. Und kann Kubernetes (oder ähnliche Plattformen) das Versprechen der unkaputtbaren Platform wirklich halten? Für wen macht Kubernetes wirklich Sinn?

## Die Idee hinter Kubernetes - Skalierbarkeit, Ausfallsicherheit und Continuous Deployment
Ursprünglich wurde die Platform von Google Entwicklern basierend auf dessen internen "Borg" System entwickelt, eine hochskalierbare Clusterverwaltung. Seit dem Initialen Release 2015 wurden bereits einige Iterationen herausgegeben und inzwischen ist Kubernetes für viele Organisationen ein essentieller Bestandteil ihrer IT-Architektur.

Um zu verstehen, ob Kubernetes das Richtige Konzept für die eigene Organisation ist, sollte man sich den Hintergrund der Entwicklung verdeutlichen. Google hatte schon früh das Konzept der horizontalen Skalierbarkeit verfolgt und maßgeblich weiterentwickelt. Grundlage ist das Paradigma, Dienste unabhängig von der darunter liegenden Infrastruktur blitzschnell über die gesamte Hardwareinfrastruktur verteilen und hochskalieren zu können und bei Nichtbedarf entsprechend wieder herunterzufahren. Das hilft nicht nur bei Lastspitzen, sondern führt gleichzeitig zu einer enormen Resilienz gegenüber Störungen. Sollten einzelne Prozesse ausfallen, können sofort neue Prozesse gestartet werden, oder aber schon laufende Prozesse den ausgefallenen übernehmen.

Neben Skalierbarkeit und Ausfallsicherheit bietet solch eine Architektur noch ein weiteres Schmankerl, inzwischen für viele der Hauptgrund für Kubernetes: Continuous Delivery oder manchmal auch Continuous Deployment. Hinter diesem Begriff versteckt sich die Möglichkeit, Applikationen und Dienste, egal ob selbstentwickelt oder zugekauft, ohne Downtime im Live-Betrieb auszurollen. Der Mechanismus dahinter ist vom Prinzip her simpel: Während der Anwender noch auf einem Service der Vorversion arbeitet, werden die neuen Versionen ausgerollt. Die Session des Anwenders mit der alten Version bleibt intakt bis dieser die Anwendung beendet. Neue Anwender, welche sich zwischenzeitlich auf den Dienst aufschalten, bekommen schon die neue Version präsentiert. Und auch der erste Anwender bekommt bei der nächsten Anmeldung die neue Version zu sehen.

Im Hintergrund wird das Deployment zum großen Teil automatisiert, so dass sich die Anwendungsverantwortlichen fast nur noch auf die Entwicklung und das Testen konzentrieren können und den Aufwand  für Releasewechsel minimieren können.




## Vertikal oder Horizontal Skalieren
Kurz zum Konzept der horizontalen Skalierbarkeit: Klassischer Weise muss man, will man mehr Leistung für seine Datenbank, Mailserver, Webserver usw. bereitstellen, die darunter liegende Hardware aufbohren. Gerade bei traditionellen Datenbanken kommt man dann irgendwo in Regionen, in denen man dutzende oder sogar hunderte Kerne, riesige Massen an RAM und Plattenkapazitäten in einer Maschine bereitstellen muss. Und damit dieser Server nicht ausfällt, was auch bedeutet, dass der Dienst ausfällt, wird alles redundant vorgehalten, was die Hardware noch teurer macht. Das nennt man *vertikale Skalierung*. Ich mache meine Hardware immer stärker.

Im Gegensatz dazu geht die *horizontale Skalierung* einen anderen Weg. Man designed seine Software so, dass die einzelnen Prozesse auf verteilter Hardware laufen kann. Anstatt einen richtig großen Rechner zu haben, benutzt man viele kleinere, billigere. Die Software muss allerdings darauf ausgelegt sein. Sie muss damit umgehen können, dass einer oder mehrere der Server jederzeit ausfallen können. Andere Server übernehmen dann. Man verlagert die Redundanz von der Hardware auf die Software. Fast alle Dienste des Internets funktionieren nach diesem Prinzip, von Google's Search über Facebook zu Netflix.

Einfach ausgedrückt verbilligt horizontale Skalierung die Hardware, erhöht aber die Komplexität der Software (das werden wir auch bei Kubernetes sehen).

## Unter der Haube: Ein Container...
Über das Architekturkonzept von Kubernetes gibt es genug gute Artikel und Einführungen, und auch hier werden wir das noch ausführlicher besprechen. Es soll erstmal ausreichen grob zu erklären, aus welchen Komponenten Kubernetes besteht und wie sich diese zusammenfügen.

Vereinfacht gesagt ist Kubernetes eine Orchestrierungsplatform für Container-Applikationen (meistens Docker-Container). Container-Applikationen sind paketierte Applikationen, die alle notwendigen Libraries und Betriebsressourcen mit sich bringen. Ein einfaches Beispiel: Wenn man sich klassisch eine Blog Engine wie zum Beispiel WordPress installieren will muss man zusätzlich zum WordPress-Paket sich auch noch einen SQL-Server und einen Webserver installieren. Letzterer muss alle notwendigen Plug-Ins mitbringen. Das ganze muss dann noch konfiguriert werden. In einem Docker-Container ist alles schon in der Art "zusammengebacken", dass man nur den Container startet und eine fertige Installation von Wordpress hat.

Für selbstgeschriebene Applikationen gilt das gleiche. Man kann in einen Docker-Container alles packen, was man zum laufen des Systems benötigt, System-Libraries usw. Das hat auch den Vorteil, dass es bei Updates des Basisbetriebssystems nicht zu unerwünschten Kompatibilitätsproblemen kommen kann, da man die Ressourcen im Container gekapselt hat.

Damit schafft man die Grundlage für das *Continuous Delivery*. Ich kann nun neben meiner aktuellen Version der Software einfach eine neue Version in einem neuen Container laufen lassen. Sobald ich mein neues Deployment geprüft habe, kann ich es freigeben und das alte abschalten. Da es gekapselt ist, muss ich nicht befürchten, dass es auf einer Produktionsumgebung nicht funktioniert, obwohl es auf der Testumgebung noch ging, weil ich vielleicht vergessen habe, die jeweiligen Betriebssysteme aneinander anzupassen. Allerdings fehlt nun noch etwas, um alles wirklich automatisieren zu können, denn bisher ist das ein manueller Prozess. Bevor wir aber dazu kommen, müssen wir uns anschauen, wie man überhaupt große Mengen von Containern orchestriert.

## ... viele Container
Kubernetes kommt dann ins Spiel, wenn man einen Service anbietet, welchen man skalieren will oder muss. Man kann leicht sehen, dass ein einzelner Container sich nicht horizontal skalieren lässt (siehe oben). Hat man aber viele gleiche Container, geht das sehr wohl. Und hier kommt Kubernetes ins Spiel. Wenn man sich zum Beispiel bei einem Paket wie WordPress anschaut, aus welchen Komponenten es besteht, kommt man auf grob gesagt drei Hauptkomponenten (es sind eigentlich viel mehr, aber ich belasse es der Einfachheit halber mal dabei): Ein Webserver, eine Datenbank und die WordPress-Engine selbst (ein großes Wort für ein paar PHP-Skripte...).

Gerade haben wir noch gelernt, dass man in einem Container all diese Komponenten zusammenfasst, um ein einfaches Deployment zu ermöglichen. Mit Kubernetes brechen wir das ganze wieder auf, und arrangieren alles neu.

Wenn man zum Beispiel eine Webseite hosten möchte, von der ich weiß, dass sie zu Stoßzeiten mehrere tausend Aufrufe pro Minute verarbeiten muss, aber zu anderen Zeiten nichts tut, dann muss man in der klassischen vertikalen Skalierung Hardware vorhalten, welche für diese Spitzenzeiten ausgelegt ist. Und wer weltweit tätig ist, muss Hardware in mehreren Regionen zur Verfügung stellen.

Mit Kubernetes kann man zum Beispiel nun den Webserveranteil herausziehen und sich einen Cluster von vielen kleinen Webservern aufbauen, welche nichts anderes tun, als die Anfragen der Anwender anzunehmen und an das Backend weiterzuleiten. Jeder dieser kleinen Container ist komplett Hardware-agnostisch und bedient nur ein paar dutzend Anfragen gleichzeitig. Wird mehr gebraucht, stehen schon weitere Container bereit oder können bei Bedarf in kürzester Zeit hochgefahren werden. Und zwar weltweit, egal ob in eigenen Datenzentren, oder aber bei Cloudprovidern.

Man muss also oft nur Teile des Dienstes horizontal skalieren, während der Rest wie gehabt bleibt. Das ist besonders dann notwendig, wenn die Software das nicht hergibt. So gibt es viele Anwendungen, welche eine relationale Datenbank benötigen, welche sich per Definition nur vertikal skalieren lässt.

## Der Preis der neuen Freiheit
Wir haben also bisher gelernt, dass Docker mein Deployment vereinfacht und Kubernetes mir Ausfallsicherheit 
Also alles super? Sollte man dann nicht gleich alles auf Kubernetes umbauen? Nein, natürlich nicht. 