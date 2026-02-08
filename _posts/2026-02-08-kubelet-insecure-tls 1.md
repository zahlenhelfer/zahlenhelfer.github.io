---
layout: post
title: "Tip: --kubelet-insecure-tls NICHT mehr nutzen"
category: kubernetes
tags:
  - blog
  - tip
  - de
  - kubernetes
  - tls
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---




## **3. Beispiel: Nginx-Pod mit `enableServiceLinks: false` in `kind`**

Wir erstellen einen **lokalen Kubernetes-Cluster mit `kind`**, starten einen **Nginx-Pod** mit deaktivierten Service-Links und machen ihn über einen **Service** erreichbar.

### **Schritt 1: `kind`-Cluster erstellen**

```sh
# Installiere kind (falls noch nicht vorhanden)
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-$(uname)-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Cluster erstellen
kind create cluster --name demo
kubectl cluster-info --context kind-demo
```

### **Schritt 2: Nginx-Pod mit `enableServiceLinks: false` erstellen**

```yaml
# nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  enableServiceLinks: false  # Deaktiviert die automatische Injektion
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

```sh
kubectl apply -f nginx-pod.yaml
```

### **Schritt 3: Service erstellen (für Zugriff)**

```yaml
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx  # Achtung: Wir müssen ein Label setzen!
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

**Problem:** Unser Pod hat kein Label `app: nginx` – also müssen wir es nachträglich setzen:

```sh
kubectl label pod nginx-pod app=nginx
kubectl apply -f nginx-service.yaml
```

### **Schritt 4: Testen**

```sh
# Port-Forwarding für lokalen Zugriff
kubectl port-forward svc/nginx-service 8080:80
```

→ Öffne [`http://localhost:8080`](http://localhost:8080/) im Browser – du solltest die **Nginx-Willkommensseite** sehen!

### **Schritt 5: Überprüfen, ob keine Service-Variablen injiziert wurden**

```sh
kubectl exec nginx-pod -- env | grep SERVICE
```

→ **Erwartetes Ergebnis:** Keine Ausgabe (weil `enableServiceLinks: false`).

---

## **4. Fazit: Wann `enableServiceLinks` nutzen oder deaktivieren?**

|**Szenario**|**Empfehlung**|
|---|---|
|**Kleiner Cluster, wenige Services**|`enableServiceLinks: true` (Standard) ist okay.|
|**Großer Cluster, viele Services**|`enableServiceLinks: false` (vermeidet Performance-Probleme).|
|**DNS-basierte Service-Discovery**|`enableServiceLinks: false` (besser `redis.default.svc.cluster.local` nutzen).|
|**Manuelle Umgebungsvariablen**|`enableServiceLinks: false` (vermeidet Konflikte).|

### **Best Practices**

✅ **Nutze DNS-Namen** statt Umgebungsvariablen (z. B. `nginx-service.default.svc.cluster.local`). ✅ **Deaktiviere `enableServiceLinks` in großen Clustern** oder wenn du Konflikte vermeiden willst. ✅ **Für Tests & Entwicklung** ist `enableServiceLinks: true` meist unproblematisch.

---

### **Weitere Ressourcen**

- [Kubernetes-Dokumentation zu `enableServiceLinks`](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#podspec-v1-core)

## Das Problem: "magische" Umgebungsvariablen in Pods
Bei meinen Trainings kommt es immer wieder vor, das wir in einem Pod eine Menge Umgebungsvariablen finden - die wir allerdings **nicht** angelegt haben.  Wenn Du in Kubernetes einen **Pod startest**, fügt der Cluster automatisch **Umgebungsvariablen für alle Services im selben Namespace** hinzu. Das sieht dann so aus:

```bash
# Beispiel: Automatisch injizierte Variablen für einen Redis-Service
REDIS_SERVICE_HOST=10.96.123.45
REDIS_SERVICE_PORT=6379
```
Aber was kann daran problematisch sein?

- **Zu viele Variablen:** In großen Clustern mit vielen Services wird die Liste der Umgebungsvariablen unübersichtlich.
- **Langsame Pod-Starts:** Kubernetes muss alle Services auflisten, was bei vielen Services zu Verzögerungen führt.
- **Konflikte:** Wenn ein Service-Name mit einer manuell gesetzten Variable kollidiert, kann das zu Fehlern führen.
- Und nicht zu vergessen, 

**Beispiel:** Falls Du eine eigene Variable `REDIS_HOST` setzt, könnte sie durch die automatische Variable überschrieben werden.

## Die Lösung: `enableServiceLinks: false`

Die einfachste Lösung ist, die automatische Injektion von Service-Variablen **komplett zu deaktivieren**. Das geht mit der Pod-Spezifikation:

```yaml
spec:
  enableServiceLinks: false  # Deaktiviert die automatische Injektion
```

### **Wann solltest du es deaktivieren?**

✔ **In großen Clustern** mit vielen Services ✔ **Wenn du DNS-basierte Service-Discovery** nutzt (z. B. `redis.default.svc.cluster.local`) ✔ **Wenn du Konflikte mit manuellen Umgebungsvariablen vermeiden willst**
## Keine Lösung: `--kubelet-insecure-tls`
Genau dafür gibt die offizielle `README.md` des `metric-server` auf [GitHub](https://github.com/kubernetes-sigs/metrics-server?tab=readme-ov-file#requirements) einen Ratschlag:
>- Kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing `--kubelet-insecure-tls` to Metrics Server)

Also schnell mal `kubectl edit deployment metrics-server -n kube-system` gestartet und `--kubelet-insecure-tls` als Argument hinzugefügt.
Aber das kann doch nicht die _richtige_ Lösung sein. Es ist doch nur ein Feld in den Zertifikaten bzw. ein Approve von der CA notwendig. Es ist aber eben Quick & Dirty und wird gerne genutzt.

## Die schnelle & saubere Lösung: `tlsBootstrap`
Da es sich bei dem Thema um die Zertifikate für das `Kubelet`handelt, sollten wir _einfach_ ein neues Zertifikat bei der Kubernetes-CA ausstellen und signieren lassen. Das ganze kann natürlich mit `openssl`-Magie passieren. Aber es geht auch noch einfacher. Es reichen diese drei Schritte: 
- Die kubelete-Config mit [`serverTLSBootstrap`](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/) erweitern
- Den Dienst `kubelet` neu starten und damit einen `csr` in Kubernetes auslösen (siehe Kubernetes-Dokumentation)
- Den `certificate signing request` in Kubernetes genehmigen

## Beispiel:  signierte Zertifikate von der Kubernetes-CA
Die config-Datei des kubelet um die folgende Zeile erweitern 

```yaml
# File: `/var/lib/kubelet/config.yaml`
serverTLSBootstrap: true
```

Jetzt noch den Dienst neu starten
```shell
systemctl restart kubelet
```

Wenn wir jetzt per `kubectl get csr` den Cluster fragen ob certificate signing request vorhanden sind, könnte die Antwort so aussehen:
```shell
NAME        AGE   SIGNERNAME                      REQUESTOR              REQUESTEDDURATION   CONDITION
csr-dnrzp   2s    kubernetes.io/kubelet-serving   system:node:k8s-node-0   <none>              Pending
```

```shell
kubectl certificate approve csr-dnrzp
```
Bitte bedenke das natürlich auf allen Nodes durchzuführen. Wenn Ihr dann bei den metrics-server-pods schaut sollte der `READY`-Status von 0/1 auf 1/1 gewechselt haben und die Fehlermeldung im Log auch verschwunden sein.
## Fazit
Manchmal ist es einfach kein Aufwand die Dinge richtig zu machen. Ja, Quick & Dirty ist schön aber lasst es uns richtig machen. 

PS: Wenn Du das schon beim Bootstrappen der Nodes beachten möchtest, schau mal in der [Kubelet Configuration (v1beta1)](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/#kubelet-config-k8s-io-v1beta1-KubeletConfiguration)-Ressource von Kubernetes nach.
