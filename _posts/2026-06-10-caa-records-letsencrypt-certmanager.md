---
title: "CAA-Records: Wer darf für Deine Domain ein Zertifikat ausstellen?"
date: 2026-06-10
categories:
  - kubernetes
tags:
  - k8s
  - dns
  - tls
  - caa
  - certmanager
  - letsencrypt
  - security
  - de
published: false
render_with_liquid: "false"
permalink: /:year/:month/:day/:title:output_ext
---
Bei meinen Trainings richten wir natürlich irgendwann den `cert-manager` ein. Es muss ja schließlich keiner mehr mit `openssl` selbstsignierte Zertifikate nutzen. Daher fällt die Wahl auf den `ClusterIssuer` für Let's Encrypt leicht. Eine Ingress-Annotation (`cert-manager.io/issuer`), und der Cluster holt sich seine TLS-Zertifikate selbstständig inkl. Ablage in einem Secret sowie automatischem Rotieren bzw. Re-Issue. Klappt prima. Dann sitzt jemand aus einem Unternehmen dabei der mir sagt "das Rating bei [sslLabs](https://www.ssllabs.com/ssltest) könnte aber besser sein.

<TODO: Bild>

Im Cluster ist alles richtig konfiguriert und sogar der Klassiker [`tlsserver`](https://letsencrypt.org/docs/profiles/#tlsserver) steht im Issuer. Das Problem liegt woanders und wir dachten es ist der CAA - __Spoiler__: ist er nicht - aber alles zu seiner Zeit. Nun erstmal was ist der CAA?!
## TL:DR;
- es darf **jede** öffentliche CA für Deine Domain ausstellen
- ein CAA-Record begrenzt das auf die, die Du wirklich nutzt
- CAA hat Zähne: Regelkonforme CAs müssen den Record seit 2017 prüfen

> ***"It's always DNS"***

Es steckt im DNS - genauer: in einem fehlenden CAA-Record. Der entscheidet, welche Certificate Authority überhaupt für Deine Domain ausstellen darf. Wer Let's Encrypt nicht einträgt, bekommt von Let's Encrypt auch kein Zertifikat.
## Das Problem: Jede CA darf für jede Domain ausstellen

Der Default im Web-PKI ist großzügig: **Jede öffentlich vertrauenswürdige CA darf für jede Domain ein Zertifikat ausstellen** - vorausgesetzt, sie hat vorher die Kontrolle über die Domain validiert ([Quelle: Let's Encrypt](https://letsencrypt.org/de/docs/caa/)). In den Trust Stores der Browser stecken Dutzende solcher CAs. Es reicht ein einziger Bug im Validierungsprozess einer CA davon, ist potenziell jede Domain betroffen ([Quelle: Let's Encrypt](https://letsencrypt.org/de/docs/caa/)). Dass jede CA für jeden Namen ausstellen kann, gilt seit jeher als eine der größten Schwächen des ganzen Systems ([Quelle: Qualys](https://blog.qualys.com/product-tech/2017/03/13/caa-mandated-by-cabrowser-forum)).

Für Deinen Cluster heißt das: `cert-manager` zeigt brav auf Let's Encrypt, aber im DNS steht nirgends, dass *nur* Let's Encrypt hier ausstellen soll. Jede andere CA könnte es trotzdem - und Du würdest es nicht mal merken.
## Die Lösung: Ein CAA-Record sagt, welche CA darf

**CAA** steht für *Certification Authority Authorization*. Es ist ein eigener DNS-Record-Typ (RR-Typ 257), 2013 erstmals standardisiert (RFC 6844); heute gilt [RFC 8659](https://datatracker.ietf.org/doc/html/rfc8659) (plus [RFC 8657](https://datatracker.ietf.org/doc/html/rfc8657) für die Erweiterungen). Mit einem CAA-Record legst Du fest, welche CAs für Deine Domain ausstellen dürfen - quaso eine Allowlist im DNS ([Quelle: Let's Encrypt](https://letsencrypt.org/de/docs/caa/)).

Diese Möglichkeit hat der Record nur, wenn die CAs ihn auch beachten - und das tun sie: Seit dem **8. September 2017** schreibt das CA/Browser Forum in seinen Baseline Requirements vor, dass jede öffentlich vertrauenswürdige CA vor der Ausstellung die CAA-Records prüfen muss. Vorher war das freiwillig ([Quelle: CA/Browser Forum, Ballot 187](https://cabforum.org/2017/03/08/ballot-187-make-caa-checking-mandatory/)). Let's Encrypt prüft CAA vor jeder einzelnen Ausstellung; der Name, den Du dafür einträgst, ist `letsencrypt.org` ([Quelle: Let's Encrypt](https://letsencrypt.org/de/docs/caa/)).

Aber ganz ehrlich an der Stelle: CAA bindet **regelkonforme** CAs. Gegen eine CA, die sich nicht an die Baseline Requirements hält oder selbst kompromittiert ist, hilft auch kein DNS-Eintrag. CAA verkleinert die Angriffsfläche, ein harter technischer Riegel ist es nicht.

Drei Tags brauchst Du im Alltag, mehr nicht:
- `issue` — welche CA überhaupt ausstellen darf. Deckt normale *und*
  Wildcard-Zertifikate ab, solange kein `issuewild` daneben steht.
- `issuewild` — nur für *Wildcard*-Zertifikate (`*.example.org`). Brauchst Du nur,
  wenn für Wildcards andere Regeln gelten sollen als sonst.
- `iodef` — wohin eine CA Verstöße melden soll, als `mailto:` oder URL. Optional,
  und nicht jede CA schickt solche Reports.

Dazu zwei Dinge, die man einmal wissen muss: Das Flag am Anfang ist fast immer `0`;`128` setzt das „critical bit" - die CA muss dann abbrechen, wenn sie den Tag nicht versteht. Und mehrere `issue`-Records sind additiv: passt *einer*, darf die CA. Die CA wertet immer den Record aus, der dem Namen am nächsten steht, und folgt dabei auch CNAMEs ([Quelle: Let's Encrypt](https://letsencrypt.org/de/docs/caa/)).

Wer es schärfer will, kann mit RFC 8657 nicht nur die CA, sondern den konkreten ACME-Account (`accounturi`) oder die erlaubte Validierungsmethode (`validationmethods`) festnageln.
## Ein Beispiel: Setzen, prüfen, von SSL Labs bestätigen lassen

**1. Den Record setzen.** In Zonen-Schreibweise sind das zwei Zeilen - nur Let's
Encrypt darf, plus eine Meldeadresse:

```text
example.org.   CAA   0 issue "letsencrypt.org"
example.org.   CAA   0 iodef "mailto:security@example.org"
```

In den meisten DNS-Oberflächen - auch bei DigitalOcean, wo meine Cluster für Trainings meistens liegen - gibt es dafür einen eigenen Record-Typ `CAA` mit den Feldern *Flags*, *Tag* und dem Wert ([Quelle: DigitalOcean](https://docs.digitalocean.com/products/networking/dns/how-to/create-caa-records/)). Auf die Registered Domain gesetzt, gilt der Record für die Domain und alle Subdomains.

**2. Prüfen mit `dig`.** Bevor Du irgendwo weiterklickst - erst nachsehen, was im DNS wirklich steht:

```bash
dig CAA example.org +short
# 0 issue "letsencrypt.org"
# 0 iodef "mailto:security@example.org"
```

**3. Von außen bestätigen lassen.** Der [SSL Server Test von Qualys SSL Labs](https://www.ssllabs.com/ssltest/) zeigt im Report eine Zeile **DNS CAA**. Steht da `Yes`, ist Dein Record veröffentlicht und von außen sichtbar - der Gegencheck, dass nicht nur Dein lokaler Resolver ihn sieht.

Und damit zurück zum Cluster: Sobald `letsencrypt.org` im CAA-Record steht, läuft die ACME-Order von `cert-manager` durch die CAA-Prüfung, und das Zertifikat wird ausgestellt. Fehlt der Eintrag, bricht die Order ab - und der Fehler aus dem Trainings-Beispiel oben wird konkret:

```bash
kubectl get certificate
# READY: False

kubectl describe order <name>
# ... CAA record for example.org prevents issuance
```
Kein Cluster-Problem. Eine fehlende Zeile im DNS.

## Kein Erfolg: die Score bleibt bei SSL Labs!
Korrekt, der CAA Eintrag hat nichts an dem Scoring verändert. Er ist nun zwar Grün, aber das reicht noch nicht. Daher haben wir weiter gesucht und sind noch ganz anderen Dinge auf den Grund gegangen. Ein alternatives Tool zum checken, andere Zertifikats-Einstellungen und das nicht jeder Ingress gleich funktioniert. Aber dazu mehr im nächsten Post. Die TLS-Reise geht weiter!
## Zusammenfassung

1. Per Default darf **jede** öffentliche CA für Deine Domain ausstellen. Ein CAA-Record schränkt das auf die ein, die Du wirklich nutzt.
2. Seit September 2017 **müssen** regelkonforme CAs CAA beachten - der Record wirkt also. Aber nur gegen regelkonforme CAs; ein harter Riegel ist er nicht.
3. Für cert-manager und Let's Encrypt heißt das konkret: `letsencrypt.org` gehört in den CAA-Record, sonst scheitert die ACME-Order mit einem CAA Fehler. Der Fix liegt im DNS, nicht im Cluster.
4. `dig CAA` zeigt Dir, was wirklich da steht; SSL Labs bestätigt es von außen. Mehr nicht und auch nicht weniger.

Denkt mal für Euch durch, wer eigentlich für Eure Domains ein Zertifikat ausstellen dürfte - und ob das wirklich nur die sind, die Ihr kennt.

---
**Quellen und Weiterlesen:**
- [Let's Encrypt — Certificate Authority Authorization (CAA), deutsch](https://letsencrypt.org/de/docs/caa/)
- [RFC 8659 — DNS Certification Authority Authorization (CAA) Resource Record](https://datatracker.ietf.org/doc/html/rfc8659)
- [RFC 8657 — CAA Record Extensions for Account URI and ACME Method Binding](https://datatracker.ietf.org/doc/html/rfc8657)
- [CA/Browser Forum — Ballot 187: Make CAA Checking Mandatory](https://cabforum.org/2017/03/08/ballot-187-make-caa-checking-mandatory/)
- [Qualys SSL Labs — SSL Server Test](https://www.ssllabs.com/ssltest/)
- [DigitalOcean — How to Manage CAA Records](https://docs.digitalocean.com/products/networking/dns/how-to/create-caa-records/)
- [SSLMate — CAA Record Generator](https://sslmate.com/caa/)
