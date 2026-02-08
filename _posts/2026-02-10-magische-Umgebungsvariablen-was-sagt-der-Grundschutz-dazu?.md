---
layout: post
title: magische Umgebungsvariablen in Pods - was sagt der IT-Grundschutz?
category: kubernetes
tags:
  - blog
  - de
  - kubernetes
  - security
  - grundschutz
  - bsi
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---
In meinem letzen Blog-Post bin ich ja auf das Thema Umgebungsvariablen in Kubernetes bzw. `enableServiceLinks: false` eingegangen. Dabei ging es ja um eine rein technische Betrachtung.

>TL;DR:
>`spec.enableServiceLinks: true` (Default) bewirkt, dass Kubernetes **automatisch Umgebungsvariablen mit Service‑Namen, IPs und Ports** aus dem Namespace in den Pod injiziert. Beispiel:

Damit gibt es also eine **Informationsoffenlegung** (interne Service‑Topologie). Zudem wird die **Angriffsfläche bei kompromittierten Pods** erhöht.
- **unnötige Kopplung an Cluster‑Details**

## Das Problem aus Sicht des IT-Grundschutz: 
Im Rahmen des IT-Grundschutz möchte ich das Thema auch für folgende Bausteine beleuchten:
- **SYS.1.6 – Containerisierung** - Minimierung der Angriffsfläche von Containern
- **APP.4.4.A3 – Identitäts‑ und Berechtigungsmanagement (B)**  
    Software‑Komponenten dürfen nur die Informationen erhalten, die für den vorgesehenen Zweck erforderlich sind.  
    → Service‑Links stellen nicht zwingend erforderliche Informationen bereit.
- **APP.4.4.A9 – Nutzung von Kubernetes Service‑Accounts (S)**  
    Pods sind so zu konfigurieren, dass nur notwendige Zugriffe und Informationen verfügbar sind.  
    → Service‑Discovery erfolgt über DNS, eine zusätzliche Bereitstellung per Umgebungsvariablen ist nicht erforderlich.
### Zusammenfassung für Prüfer:innen
Die Deaktivierung von `enableServiceLinks` dient der Reduktion unnötiger Informationsoffenlegung innerhalb von Kubernetes‑Pods und setzt die Anforderungen des BSI IT‑Grundschutzes (APP.4.4 Kubernetes, SYS.1.6 Containerisierung) um. Die Maßnahme unterstützt das Least‑Privilege‑Prinzip, reduziert die Angriffsfläche und ist Bestandteil der standardisierten und auditierbaren Kubernetes‑Baseline.
## Das Problem: Kubernetes "magische" Umgebungsvariablen  und der IT-Grundschutz
Bei meinen Trainings kommt es immer wieder vor, das wir in einem Pod eine Menge Umgebungsvariablen finden. Diese haben wir allerdings **nicht** angelegt und sie sind auch nicht im _Dockerfile_ erstellt worden. Die Erklärung ist ganz einfach. Wenn Du in Kubernetes einen **Pod** startest, fügt der Cluster automatisch **Umgebungsvariablen** für **alle Services** im **selben Namespace** hinzu. Dieses Verhalten kannst Du auch in der Kubernetes Dokumentation [ Accessing the Service](https://kubernetes.io/docs/tutorials/services/connect-applications-service/#accessing-the-service) nachlesen. Das kann dann z.B. so aussehen:

```bash

# Beispiel: Automatische Variablen für einen NGINX-Service
NGINX_SERVICE_SERVICE_HOST=10.96.117.78
NGINX_SERVICE_SERVICE_PORT=80
NGINX_SERVICE_PORT_80_TCP_ADDR=10.96.117.78
NGINX_SERVICE_PORT_80_TCP=tcp://10.96.117.78:80
NGINX_SERVICE_PORT_80_TCP_PROTO=tcp
NGINX_SERVICE_PORT_80_TCP_PORT=80
NGINX_SERVICE_PORT=tcp://10.96.117.78:80
```
Wenn Du mehrere Services in einem Namespace hast, wird die Liste natürlich länger. Aber was kann daran so problematisch sein?

- **Zu viele Variablen:** In großen Clustern mit vielen Services wird die Liste der Umgebungsvariablen unübersichtlich.
- **Langsames starten von Pods:** Kubernetes muss alle Services auflisten, was bei vielen Services zu Verzögerungen führen kann.
- **Konflikte:** Wenn ein Service-Name mit einer manuell gesetzten Variable kollidiert, kann das zu Fehlern führen.
- Wenn dazu noch **Auto-Scaling** via HPA oder KEDA dazu kommt, müssen für jeden Pod diese Variablen erzeugt werden. Das kostet BootStrap-Zeit beim skalieren.
- Im Rahmen von **Compliance und Sicherheit** geben wir damit potenziellen Angreifern zusätzliche Informationen über den Cluster bzw. die Services. Das kann nicht gewollt sein. Daher werde ich insbesondere dieses Thema in einem separaten Post für den IT-Grundschutz aufbereiten bzw. einordnen.

Jetzt werden einige vielleicht argumentieren, "aber der Workload muss doch wissen wo z.B. ein Redis-Cache oder eine Message-Queue ist!". Das wäre hiermit über die Umgebungsvariablen super dynamisch möglich! Ja, aber wir nutzen doch meistens das **DNS-basierte Service-Discovery** wie in unserem Beispiel also: `nginx-service.default.svc.cluster.local`. Mehr dazu findet Ihr in der Kubernetes Doku zu [SRV Records](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#srv-records) Damit brauchen wir diese Umgebungsvariablen nun wirklich nicht mehr.
## Die Lösung: `enableServiceLinks: false`
Die einfachste Lösung diese "Magie" los zu werden ist, dieses Verhalten **komplett zu deaktivieren**. Das geht mit nur einer Zeile in der `spec` des Pods. :

```yaml
spec:
  enableServiceLinks: false
```

> Wichtige Anmerkung: Wer das direkt ausprobiert, wird sehen das es **immer** die Umgebungsvariablen `KUBERNETES_SERVICE_*` und `KUBERNETES_PORT*` gibt. Dieses Verhalten ist fest in Kubernetes kodiert. Ein `enableServiceLinks:false` ändert es auch nicht!
## Beispiels : Pods mit und ohne `enableServiceLinks: false`
Jetzt starten wir mit einem Test-Namespace für die Anwendung und zwei **Nginx-Pods**. Dabei jeweils einmal mit und einmal ohne Service-Links in der Definition. Dazu kommt abschliessend noch ein Service.
### Schritt 1: Vorbereitung der Ressourcen für das Beispiel

```yaml
# ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test-links
```

```yaml
# nginx-pods.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-with-links
  namespace: test-links
spec:
  enableServiceLinks: true # default
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-without-links
  namespace: test-links
  labels:
    app: nginx
spec:
  enableServiceLinks: false
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

```yaml
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: test-links
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

```sh
kubectl apply -f ns.yaml
kubectl apply -f nginx-pods.yaml
kubectl apply -f nginx-service.yaml
```
### Schritt 2: Überprüfen, ob Service-Variablen "magisch" erstellt wurden

Jetzt testen wir, wie die Ausgabe beim standard (`true`) Verhalten.
```sh
kubectl -n test-links exec nginx-pod-with-links -- env
```
Erwartetes Ergebnis: `NGINX`- und `KUBERNETES`-Variablen mit `SERVICE` und `PORT` Werten.
```sh
NGINX_SERVICE_SERVICE_PORT=80
NGINX_SERVICE_PORT_80_TCP=tcp://10.96.117.78:80
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
NGINX_SERVICE_PORT_80_TCP_PROTO=tcp
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
NGINX_SERVICE_PORT_80_TCP_PORT=80
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NGINX_SERVICE_SERVICE_HOST=10.96.117.78
NGINX_SERVICE_PORT=tcp://10.96.117.78:80
NGINX_SERVICE_PORT_80_TCP_ADDR=10.96.117.78
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
```

Nun die Gegenprobe mit dem anderen Pod bei dem `enableServiceLinks: false` gesetzt wurde.
```sh
kubectl -n test-links exec nginx-pod-without-links -- env
```
Erwartetes Ergebnis: Hier finden sich nur die `KUBERNETES`-Variablen.
```sh
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
```
## Fazit: Wann `enableServiceLinks` nutzen oder deaktivieren?

Für Tests & Entwicklung ist der Standard mit `enableServiceLinks: true` meist unproblematisch. Auch bei kleineren Cluster und wenigen Workloads kann es sicherlich vernachlässigt werden. Wenn zudem jede Anwendung ihren eigenen Namespace bekommt (Best Practice) - ist die Anzahl der Variablen meist überschaubar. **Aber** wird dieses Feature ermöglicht uns Information über andere Teile der Anwendung zu Sammeln obwohl wir nur Zugriff auf diesen Container haben. Das ist bei vielen Produktionssystemen bestimmt nicht gewollt. Fragt auch gerne mal Euer SOC zu dem Thema. Die Alternative wird ja meist eh schon genutzt. Das **DNS-basierte Service-Discovery** (nginx-service.default.svc.cluster.local).  Daher frei nach Löwenzahn - "einfach mal abschalten"