---
layout: post
title: BSI Grundschutz - APP.4.4.A2 Planung der Automatisierung mit CI/CD (B) - Teil 3
category: kubernetes
tags:
  - blog
  - de
  - bsi
  - grundschutz
  - kubernetes
  - argocd
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---

# BSI Grundschutz - APP.4.4.A2 Planung der Automatisierung mit CI/CD (B) - Teil 3

>DISCLAIMER: Ich bin weder Auditor noch Anwalt - daher keine Gewähr!

Weiter geht es heute mit dem nächsten Baustein. Dieser erscheint eigentlich vom Text zu kurz für einen Blogpost, aber da steckt eine Menge drin. Also legen wir gleich los.
### APP.4.4.A2 Planung der Automatisierung mit CI/CD (B)
>Wenn eine Automatisierung des Betriebs von Anwendungen in Kubernetes mithilfe von CI/CD stattfindet, DARF diese NUR nach einer geeigneten Planung erfolgen. 

Du willst ein CI/CD Tool wie Flux oder ArgoCD für die Automatisierung nutzen. Das ist prima. Denn diese Tools stellen sicher, dass dein Cluster auch nach dem Installieren einer Anwendung mittels Manifesten oder Helm-Chart auch wirklich den gewünschten Stand behält.

>Die Planung MUSS den gesamten Lebenszyklus von Inbetrieb bis Außerbetriebnahme inklusive Entwicklung, Tests, Betrieb, Überwachung und Updates umfassen. 

Lebenszyklus also von der Installation über Updates bis zur Stillegeung. Da hilft das Nutzen von Helm-Charts doch sehr. Die Anwendung ist damit paketiert und kann installiert, aktualisiert und wieder deinstalliert werden.

>Das Rollenund Rechtekonzept sowie die Absicherung von Kubernetes Secrets MÜSSEN Teil der Planung sein.

Und auch hier wieder zwei Teile. Der RBAC Teil das dass Rechte und Rollenkonzept beinhaltet. Und die `Secrets` die z.B. per External-Secret-Operator oder Sealed Secrets oder mit etcd-Encryption gesichert werden können. 
