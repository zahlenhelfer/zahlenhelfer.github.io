---
layout: post
title: 'Tip:  enableServiceLinks - "magischen" Umgebungsvariablen in Pods'
category: kubernetes
tags:
  - blog
  - tip
  - de
  - kubernetes
  - pod
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---
## Das Problem: "magische" Umgebungsvariablen in Pods
Bei meinen Trainings kommt es immer wieder vor, das wir in einem Pod eine Menge Umgebungsvariablen finden. Diese haben wir allerdings **nicht** angelegt.  Die Erklärung ist ganz einfach. Wenn Du in Kubernetes einen **Pod** startest, fügt der Cluster automatisch **Umgebungsvariablen** für **alle Services** im **selben Namespace** hinzu. Dieses Verhalten kannst Du auch in der [Kubernetes Dokumentation](https://kubernetes.io/docs/tutorials/services/connect-applications-service/#accessing-the-service) nachlesen. Das kann dann z.B. so aussehen:

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
- **Langsames Pod starten:** Kubernetes muss alle Services auflisten, was bei vielen Services zu Verzögerungen führen kann.
- **Konflikte:** Wenn ein Service-Name mit einer manuell gesetzten Variable kollidiert, kann das zu Fehlern führen.
- Wenn dazu noch **Auto-Scaling** via HPA oder KEDA dazu kommt, müssen für jedem Pod diese Variablen erzeugt werden. Das kostet BootStrap-Zeit beim Deployment.
- Im Rahmen von **Compliance und Sicherheit** werde ich das Thema in einem separaten Post für den IT-Grundschutz aufbereiten.

> Was nicht bekannt ist, kann auch schwerer gehackt werden. :)

Aber der Workload muss doch wissen wo z.B. ein Redis-Cache oder eine MessageQueue ist. Das wäre hiermit ja super dynamisch! Ja, aber wir nutzen doch meistens das **DNS-basierte Service-Discovery** wie in unserem Beispiel: `nginx-service.default.svc.cluster.local`. Mehr dazu findet Ihr in der Kubernetes Doku zu [SRV Records](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#srv-records) Damit brauchen wir diese Umgebungsvariablen nicht.
## Die Lösung: `enableServiceLinks: false`
Die einfachste Lösung ist, dieses Verhalten **komplett zu deaktivieren**. Das geht mit folgender Pod-Spezifikation:

```yaml
spec:
  enableServiceLinks: false
```

> Wichtige Anmerkung: Wer das direkt ausprobiert, wird sehen das es **immer** die Umgebungsvariablen `KUBERNETES_SERVICE_*` und `KUBERNETES_PORT*` gibt. Dieses Verhalten ist fest in Kubernetes kodiert. Ein `enableServiceLinks:false` ändert es auch nicht!
## Beispiel: Pod mit `enableServiceLinks: false`
In diesem Beispiel starten wir einen **Nginx-Pod** mit deaktivierten Service-Links und machen ihn über einen **Service** erreichbar.
### Schritt 1: Nginx-Pods mit  und ohne `enableServiceLinks: false` erstellen

```yaml
# nginx-pods.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-with-links
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
kubectl apply -f nginx-pods.yaml
kubectl apply -f nginx-service.yaml
```

### Schritt 2: Überprüfen, ob Service-Variablen "magisch" erstellt wurden

Jetzt testen wir, wie die Ausgabe beim standard Verhalten ist:
```sh
kubectl exec nginx-pod-with-links -- env | grep SERVICE
```

→ **Erwartetes Ergebnis:**

Nun die Gegenprobe mit dem anderen Pod bei dem `enableServiceLinks: false` gesetzt wurde:
```sh
kubectl exec nginx-pod-without-links -- env | grep SERVICE
```

→ **Erwartetes Ergebnis:**

Nur die Kubernetes-Variablen sollten ausgegeben werden (weil `enableServiceLinks: false`).

## Fazit: Wann `enableServiceLinks` nutzen oder deaktivieren?

Für Tests & Entwicklung ist der Standard mit `enableServiceLinks: true` meist unproblematisch. Auch bei kleineren Cluster und wenigen Workloads. Wenn zudem jede Anwendung ihren eigenen Namespace bekommt, ist die Anzahl der Variablen meist überschaubar. _Aber_ wird dieses Feature bzw. die Infomation der Variablen auch wirklich von der Anwendungen genutzt? Wir haben doch noch das **DNS-basierte Service-Discovery** (nginx-service.default.svc.cluster.local).