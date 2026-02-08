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
In meinem letzen [Blog-Post](https://zahlenhelfer.github.io/2026/02/08/enableServiceLinks-oder-magische-Umgebungsvariablen.html) bin ich auf das Thema Umgebungsvariablen in Kubernetes bzw. `enableServiceLinks: false` eingegangen. Dabei ging es ja um eine rein technische Betrachtung des Thema. Nochmal kurz zusammengefasst:
>TL;DR:
>`spec.enableServiceLinks: true` (Default) bewirkt, dass Kubernetes **automatisch Umgebungsvariablen mit Service‑Namen, IPs und Ports** aus dem Namespace in den Pod injiziert. 
>
>Beispiel:
>`kubectl exec nginx-pod -- env`
>`NGINX_SERVICE_SERVICE_HOST=10.96.117.78`
>`NGINX_SERVICE_SERVICE_PORT=80`
## Das Problem aus Sicht des IT-Grundschutz
Durch das "magische" injitzieren von Umgebungsvariablen in Pod mit Service und Port Informationen gibt es eine ungewollte **Informationsoffenlegung** (interne Service‑Topologie). Das ist sicherheitsrelevant, denn Umgebungsvariablen sind:
- **für jeden Prozess im Container lesbar**
- häufig **Teil von Debug‑Ausgaben / Dumps**
- leicht **exfiltrierbar bei Kompromittierung**
Damit werden interne Architektur‑ und Netzwerkdetails preisgegeben. Zudem wird natürlich die **Angriffsfläche bei kompromittierten Pods** erhöht. Das ganze bewirkt also eine **unnötige Kopplung an Cluster‑Details**. Oder ganz direkt - zu viel Information in einem Pod die meist nie durch den Workload genutzt wird. Anwendungen nutzen das **DNS-basierte Service-Discovery**, also über Namen wie z.B. `nginx-service.default.svc.cluster.local`. Mehr dazu findet Ihr in der Kubernetes Doku zu [SRV Records](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#srv-records). Was können wir nun also im IT-Grundschutz dazu finden?
## Auslegungen
Es gibt keinen einzelnen BSI‑Grundschutz‑Baustein, der `enableServiceLinks: false` explizit fordert, aber diese Maßnahme lässt sich klar und sauber aus dem Baustein _APP.4.4 Kubernetes_ ableiten, unterstützt durch _SYS.1.6 Containerisierung_.
### SYS.1.6 – Containerisierung
Die SYS.1.6. betrachtet Container als **potenziell kompromittierbar** und fordert daher eine **Minimierung von Informationen, Rechten und Abhängigkeiten im Container**. Ein kompromittierter Container mit Service Links:
- kennt interne Services
- kennt erreichbare Ports
- kann gezielt lateral agieren
➡️ Durch das Setzen von `enableServiceLinks: false` werden
- verwertbare Informationen reduziert
- Seitwärtsbewegungen erschwert
### APP.4.4.A3 – Identitäts‑ und Berechtigungsmanagement (B)
Software‑Komponenten dürfen nur die Informationen erhalten, die für den vorgesehenen Zweck erforderlich sind.
➡️ Service‑Links stellen **nicht notwendige Informationen** bereit, wenn DNS genutzt wird.
### APP.4.4.A9 – Nutzung von Kubernetes Service‑Accounts (S)
Pods sind so zu konfigurieren, dass nur notwendige Zugriffe und Informationen verfügbar sind.  
➡️ Service‑Discovery erfolgt über DNS, eine zusätzliche Bereitstellung per Umgebungsvariablen ist nicht erforderlich.

### APP.4.4.A13 – Automatisierte Auditierung der Konfiguration (H)
Sicherheitsrelevante Kubernetes‑Konfigurationen sind standardisiert und prüfbar umzusetzen. 
➡️ Die Deaktivierung von Service‑Links entspricht etablierten Sicherheitsbenchmarks (z. B. CIS Kubernetes Benchmark). Zudem 
## Praxis-Maßnahme : Kyverno-Policy
Der Kubernetes‑Cluster implementiert eine definierte und geprüfte Sicherheitsbaseline auf Basis von Policy‑as‑Code. Die Einstellung `enableServiceLinks: false` wird im Rahmen des Admission‑Controllers im Audit‑Modus überwacht. Hiermit werden Abweichungen von der Soll‑Konfiguration frühzeitig im Betrieb erkannt, damit sind nachvollziehbare Nachweise zur Erfüllung der Anforderungen des BSI‑Bausteins SYS.1.6 Containerisierung sowie der ISO/IEC 27001:2022 unter Berücksichtigung der Intention des CIS Kubernetes Benchmarks bereitzustellen.
```yaml
# disabele-serviceLinks.yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
  metadata:
    name: require-disable-service-links
    annotations:
      policies.kyverno.io/title: Disable Kubernetes Service Links
      policies.kyverno.io/category: Security Baseline
      policies.kyverno.io/severity: medium
      policies.kyverno.io/subject: Pod
      policies.kyverno.io/description: >
        Ensures that Kubernetes Pods disable Service Links to reduce implicit exposure of service metadata in accordance with BSI SYS.1.6, ISO/IEC 27001:2022 and CIS Kubernetes Benchmark intent.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: service-links-must-be-disabled
      match:
        resources:
          kinds:
            - Pod
	  exclude:
		resources:
		  namespaces:
			- kube-system
			- kyverno
	  validate:
		message: >
		  Pod must explicitly set spec.enableServiceLinks: false to comply with the container security baseline (SYS.1.6).
		pattern:
		  spec:
			enableServiceLinks: false
```
## Zusammenfassung für Prüfer:innen
Die Deaktivierung von `enableServiceLinks` dient der Reduktion unnötiger Informationsoffenlegung innerhalb von Kubernetes‑Pods und setzt die Anforderungen des BSI IT‑Grundschutzes (APP.4.4 Kubernetes, SYS.1.6 Containerisierung) um. Die Maßnahme unterstützt das Least‑Privilege‑Prinzip, reduziert die Angriffsfläche und ist Bestandteil der standardisierten und auditierbaren Kubernetes‑Baseline.