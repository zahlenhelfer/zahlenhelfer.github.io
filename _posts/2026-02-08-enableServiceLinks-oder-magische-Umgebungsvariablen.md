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
- **Langsame Pod-Starts:** Kubernetes muss alle Services auflisten, was bei vielen Services zu Verzögerungen führt.
- **Konflikte:** Wenn ein Service-Name mit einer manuell gesetzten Variable kollidiert, kann das zu Fehlern führen.
- Im Rahmen von Compliance und Sicherheit werde ich das Thema in einem separaten Post für den IT-Grundschutz beleuchten.

> Wichtige Anmerkung: Wer das direkt ausprobiert wird sehen, das es **immer** die Umgebungsvariablen `KUBERNETES_SERVICE_*` und `KUBERNETES_PORT*` gibt. Dieses Verhalten ist fest in Kubernetes kodiert. Ein `enableServiceLinks:false` ändert dies auch nicht!
## Die Lösung: `enableServiceLinks: false`
Die einfachste Lösung ist dieses Verhalten **komplett zu deaktivieren**. Das geht mit folgender Pod-Spezifikation:

```yaml
spec:
  enableServiceLinks: false
```

## Beispiel: Pod mit `enableServiceLinks: false`
In diesem Beispiel starten wir einen **Nginx-Pod** mit deaktivierten Service-Links und machen ihn über einen **Service** erreichbar.
### Schritt 1: Nginx-Pod mit `enableServiceLinks: false` erstellen

```yaml
# test-ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test-links
```

```yaml
# nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
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
kubectl apply -f nginx-pod.yaml
kubectl apply -f nginx-service.yaml
```

### Schritt 2: Überprüfen, ob keine Service-Variablen "magisch" erstellt wurden

```sh
kubectl exec nginx-pod -- env | grep SERVICE
```

→ **Erwartetes Ergebnis:** Keine Ausgabe (weil `enableServiceLinks: false`).

## Fazit: Wann `enableServiceLinks` nutzen oder deaktivieren?**

| **Szenario**                         | **Empfehlung**                                                                 |
| ------------------------------------ | ------------------------------------------------------------------------------ |
| **Kleiner Cluster, wenige Services** | `enableServiceLinks: true` (Standard) ist okay.                                |
| **Großer Cluster, viele Services**   | `enableServiceLinks: false` (vermeidet Performance-Probleme).                  |
| **DNS-basierte Service-Discovery**   | `enableServiceLinks: false` (besser `redis.default.svc.cluster.local` nutzen). |
| **Manuelle Umgebungsvariablen**      | `enableServiceLinks: false` (vermeidet Konflikte).

### **Best Practices**

✅ **Nutze DNS-Namen** statt Umgebungsvariablen (z. B. `nginx-service.default.svc.cluster.local`). 
✅ **Deaktiviere `enableServiceLinks` in großen Clustern** oder wenn du Konflikte vermeiden willst. 
✅ **Für Tests & Entwicklung** ist `enableServiceLinks: true` meist unproblematisch.
### **Weitere Ressourcen**

- [Kubernetes-Dokumentation zu `enableServiceLinks`](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#podspec-v1-core)
