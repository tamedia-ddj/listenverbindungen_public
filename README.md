# Listenverbindungen



![Listenverbindungen in a nutshell](https://interaktiv.tagesanzeiger.ch/2019/listenverbindungen/so-funktioniert-eine-listenverbindung.svg)

## Analyse der Wirkung von Listenverbindungen an der Schweizer Nationalratswahlen 2019

Im Nachgang zu den am 20. Oktober 2019 stattgefundenen Nationalratswahlen führte das Tamedia-Datenteam im Rahmen der Nachwahlanalysen eine Untersuchung der Listenverbindungen durch. Die Resultate wurden am 29. Oktober 2019 im Artikel "Wahllisten-Poker: Wem er nützte, wer verlor" veröffentlicht.

**Datenquelle(n)**: Bundesamt für Statistik ([opendata.swiss](https://opendata.swiss/de/dataset/eidg-wahlen-2019))

**Artikel**: [Wahllisten-Poker: Wem er nützte, wer verlor](https://www.tagesanzeiger.ch/schweiz/wahlen/so-haben-die-mitteparteien-abgeraeumt/story/10506290)

**Code**: [main.ipynb](https://github.com/tamedia-ddj/listenverbindungen_public/blob/master/main.ipynb)



## Beschreibung
Um die Gewinner und Verlierer der Listenverbindungen zu bestimmen, wurden mehrere Szenarien simuliert. Als Basis dienten die offiziellen Zahlen des Bundesamts für Statistik zu den eingereichten Wahllisten, den jeweiligen Listenverbindungen und den abgegebenen Stimmen nach Listen und Kantonen.

In einem ersten Szenario wurden beim Berechnen der Sitzverteilung im Nationalrat überparteiliche Listenverbindungen komplett ignoriert. Differenzen zum realen Wahlresultat ergeben daraus die Bilanz über die Wirkung der Listenverbindungen.

Ausgehend von den existierenden Listenverbindungen wurden für weitere Szenarien jeweils bestimmte Listenverbindungen gestrichen oder neue hinzugefügt. Die daraus resultierenden Sitzverteilungen ermöglichen Aussagen über die Wirkung alternativer Allianzen.

Wenn eine Partei in einem Kanton mehrere Listen präsentiert hat (zum Beispiel Listen der Jungpartei oder eigene Seniorenlisten) wurden deren Stimmen in allen Szenarien zusammengefasst.



## Berechnungen

**Datengrundlage**

Das Bundesamt für Statistik stellt bereits am Abend des Wahltages die Resulate der Wahlen auf opendata.swiss zur öffentlichen Verfügung. Neben Informationen zu Wahlbeteiligung, dem Abschneiden der Parteien auf Gemeinde- und Kantonsebene und den einzelnen Kandidierenden, werden die Wahl-Resulate auch auf Ebene der [kantonalen Wahllisten](https://www.bfs.admin.ch/bfsstatic/dam/assets/9386464/master) angeboten. Nachfolgend werden die zentralen Variabeln dieser "Listen"-Tabelle hervorgehoben:

Variable | Beschreibunng
--- | --- 
`...` | ...
`kanton_nummer ` | Eindeutige Kennziffer für die Kantone
`liste_bezeichnung ` | Bezeichnung der jeweiligen Liste (Listentitel)
`...` | ...
`liste_unterlistenverbindung ` | Eine pro Kanton eindeutige Kennzeichung einer Unterlistenverbindung. Bestehend aus Buchstabe der Listenverbindung und aufsteigender Nummer für Unterlistenverbindung. (z.B.: `C2`)
`liste_verbindung ` | Eine pro Kanton eindeutige Kennzeichung einer Listenverbindung. Bestenhend aus aufsteigenden Buchstaben. (z.B.: `C`)
`...` | ...
`partei_id ` | Eindeutige Kennziffer für die (Mutter-)Partei der jeweiligen Liste
`stimmen_liste ` | Absolute Anzahl Stimmen, die die Liste im Kanton erhalten hat.


 
**Szenario 1: Sitzverteilung ohne Listenverbindungen:**

In einem ersten Szenario, dem Kern der Untersuchung, wird berechnet, wie die Sitzverteilung aussähe, gäbe es keine Listenverbindungen. Dazu werden pro Kanton (`kanton_nummer`) die Anzahl Stimmen (`stimmen_liste`) von allen Listen die der selben Partei (`partei_id`) angehören addiert. Gemäss dieser Stimmeanteile werden pro Kanton die ihm zustehenden Nationalratssitze neu verteilt. Dazu werden erst für jedes Vielfache der benötigten Stimmenzahl für einen Sitz den Parteien Sitze zugewiesen. Danach werden die Restmandate verteilt. (Anleitung: [Bundeskanzlei](https://www.bk.admin.ch/bk/de/home/politische-rechte/nationalratswahlen/nationalratswahlen-2019.html)) 

<!--Vollmandate


```
for i, row in tqdm(parteien.iterrows(), total=len(parteien)):
    parteien.loc[i, "sitze_parteien"] = math.floor(row['stimmen_liste'] / row['verteilungszahl'])
    parteien.loc[i, "reststimmen"] = row['stimmen_liste'] % row['verteilungszahl']
```
Restmandate

```
parteien["restmandate"] = 0

for kanton in tqdm(parteien.kanton_nummer.unique()):
    restmandate = parteien[parteien["kanton_nummer"] == kanton].anzahl_gewaehlte.sum() - parteien[parteien["kanton_nummer"] == kanton].sitze_parteien.sum()
    
    while restmandate > 0:
        temp = parteien[parteien["kanton_nummer"] == kanton]
        temp["verteilungrest"] = temp['stimmen_liste'] / (temp['sitze_parteien'] + temp['restmandate'] + 1)
        idx = temp.sort_values(["verteilungrest"], ascending=False).index[0]
        parteien.loc[idx, "restmandate"] += 1
        restmandate -= 1
```-->

Zum Schluss von *Szenario 1* wird die Differenz zur realen Verteilung der Nationalratssitze berechnet,  exportiert (`diff_keine_listenverbindungen.csv` - siehe Output-Files) und zur Visualisierung weitergegeben.


**Weitere Szenarien**

Für die weiteren Szenarien wurden Listen- und Unterlistenverbindungen berücksichtigt. Mit der Funktion `calc_seats()`, die als Input die Listeninformationen (in der Form wie sie das BFS liefert) erwartet, wird unter Berücksichtigung aller Verbindungen die Sitzverteilung pro Kanton berechnet. Als Antwort liefert die Funktion eine Tabelle, die pro Kanton und Partei die gewonnenen Nationalratssitze ausweist.

Um nun die Auswirkungen von neuen Kombinationen von Listenverbindungen auf die Zusammensetzung des Parlaments zu ermitteln, kann die ursprüngliche "Listen"-Tabelle modifiziert und erneut der  `calc_seats()`-Funktion übergeben werden. Durchgeführte Modifikationen der "Listen"-Tabelle sind:


* FDP tritt in allen Kantonen den bestehenden SVP-Listenverbindungen bei. (falls keine Listenverbindung vorhanden wird eine erstellt)
* Listenverbindungen von SP und Grünen werden in allen Kantonen aufgelöst. (Die Grünen verlassen die Listenverbindungen in denen SP vertreten ist)
* FDP tritt in allen Kantonen den bestehenden CVP-Listenverbindungen bei. (falls keine Listenverbindung vorhanden wird eine erstellt)
* Die breiten Mitte-Allianzen (EVP / CVP / BDP / GLP) werden aufgelöst.
* FDP tritt aus allen Listenverbindungen aus
* EDU tritt aus allen Listenverbindungen aus

zum Test ausserdem:

* **Szenario 0**: Alles bleibt wie es ist.

Wird das Argument `print_output = True` der Funktion `calc_seats()` übergeben, lässt sich die Berechung der Sitzverrteilung über die verschiedenen Ebenen (Listen, Listenverbindung, Unterlistenverbindung) Schritt für Schritt nachvollziehen.



## Output-Files

### output/diff\_keine\_listenverbindungen.csv

Variable | Beschreibunng
--- | --- 
`partei_bezeichnung_kurz_de ` | Parteibezeichnung Kurzform
`sitze_diff ` | Differenz zur realen Sitzverteilung im Nationalrat - negative Werte bedeuten, dass in einer Wahl ohne Listenverbindung die jeweilige Partei mehr Sitze erhalten hätte


### output/svpfdp\_listenverbindung.csv

Variable | Beschreibunng
--- | --- 
`parteiname ` | Parteibezeichnung Kurzform
`sitze ` | Anzahl Sitze im Nationalrat, die der jeweiligen Partei auf Basis der modifizierten "Listen"-Tabelle zustehen

### output/spgps\_listenverbindung.csv

Variable | Beschreibunng
--- | --- 
`parteiname ` | Parteibezeichnung Kurzform
`sitze ` | Anzahl Sitze im Nationalrat, die der jeweiligen Partei auf Basis der modifizierten "Listen"-Tabelle zustehen

### output/ohne\_mitteallianz\_listenverbindung.csv

Variable | Beschreibunng
--- | --- 
`parteiname ` | Parteibezeichnung Kurzform
`sitze ` | Anzahl Sitze im Nationalrat, die der jeweiligen Partei auf Basis der modifizierten "Listen"-Tabelle zustehen

### output/fdpcvp\_listenverbindung.csv

Variable | Beschreibunng
--- | --- 
`parteiname ` | Parteibezeichnung Kurzform
`sitze ` | Anzahl Sitze im Nationalrat, die der jeweiligen Partei auf Basis der modifizierten "Listen"-Tabelle zustehen



## Lizenz

*Listenverbindungen Public* is free and open source software released under the permissive MIT License.

