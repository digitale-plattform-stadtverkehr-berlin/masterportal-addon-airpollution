Entwicklung Masterportal Addon Airpollution (gfiTheme)
======================================================

[Masterportal](https://bitbucket.org/geowerkstatt-hamburg/masterportal/src/dev/) Airpollution-Addon zur Verwendung in der "Digitale Plattform Stadtverkehr - Berlin"-Plattform.

Das Plugin dient zur Darstellung von Luftschadstoff-Prognosen.

Getestet mit Masterportal Version 2.30.0


## Airpollution Parameter

Werden an das Airpollution-Plugin als gfi Theme-Parameter angegeben.

**type**

String: Name des Schadstoffs, für den die entsprechendne Luftschadstoffprognose angezeigt werden sollen. Wird im "title" des Diagramms dargestellt.

**datastream**

Array von Arrays, wobei jedes Unterarray aus einem Datums- und einem numerischen Wert (der jeweiligen Prognose des Luftschadstoffwertes zum entsprechenden Zeitpunkt) besteht. Z.B.: 
```
[ [ "2023-02-17T05:00:00", 15.1 ], [ "2023-02-17T06:00:00", 20.3 ], [ "2023-02-17T07:00:00", 24 ], [ "2023-02-17T08:00:00", 24.4 ], 
... 
]
```

**uom** (unit of measurement)

String: Bezeichnung der Maßeinheit; wird an der y-Achse angezeigt.

**threshold**

Array von Objekten zur Definition der Schwellenwerte:
Jedes Objekt besteht aus der unteren Messwert-Grenze des jeweiligen Bereichs ("value"), einer zugeordneten Balken-Farbe ("color") und einer Bewertung der entsprechenden Schdstoffstufe ("rating"). Die einzelene Objekte eintsprechen den Balken des Diagramm.


### Vollständiges Beispiel Airpollution-Parameter:

```
gfiTheme": {
  "name": "airpollution",
  "params": {
    "type": "NO₂",
    "datastream": "no2_json",
    "uom": "µg/m³",
    "threshold": [
      {"value": 0, "color": "#5ad14f", "rating": "Sehr gut"},
      {"value": 12.5, "color": "#9bd14f", "rating": "Gut"},
      {"value": 25, "color": "#eefa00", "rating": "Befriedigend"},
      {"value": 37.5, "color": "#faa100", "rating": "Ausreichend"},
      {"value": 50, "color": "#f24726", "rating": "Schlecht"},
      {"value": 100, "color": "#c00000", "rating": "Sehr schlecht"}
    ]
  }
},
```
