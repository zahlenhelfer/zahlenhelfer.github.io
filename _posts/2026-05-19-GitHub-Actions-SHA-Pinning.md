---
title: "Tip: GitHub Actions auf SHA pinnen - warum jetzt und nicht irgendwann"
date: 2026-05-19
categories:
  - github
tags:
  - gha
  - github
  - security
  - devsecops
published: false
render_with_liquid: "false"
permalink: /:year/:month/:day/:title:output_ext
---

## Das Problem: `@v4` ist kein fixer Punkt - sondern ein beweglicher Zeiger

In nahezu jedem Workflow, den ich in den letzten Jahren in meinen Trainings gesehen habe, steht so etwas:

```yaml
- uses: actions/checkout@v4
- uses: docker/build-push-action@v5
- uses: hashicorp/setup-terraform@v3
```

Sieht ordentlich aus, ist gepflegt, "wir haben ja eine Version drin". Das Dumme ist nur: `@v4` ist kein unveränderliches Release. Es ist ein **Git-Tag**, und ein Tag ist nichts anderes als ein Zeiger auf einen Commit. Wer Schreibrechte auf das Action-Repo hat - der Maintainer, ein kompromittierter Account, ein übernommener PAT - kann den Tag jederzeit auf einen anderen Commit verschieben. Dein nächster Pipeline-Run zieht dann automatisch den neuen Stand. Ohne PR, ohne Review, ohne dass Du es merkst.

## Kein theoretisches Problem - März 2025 - `tj-actions/changed`-files

Im März 2025 wurde genau das real: Bei der weit verbreiteten Action `tj-actions/changed-files` wurden **alle Versions-Tags umgeschrieben** und auf einen Commit mit Schadcode umgebogen. Rund 23.000 Repositories haben diese Action genutzt. Jeder Workflow, der während des Zeitfensters lief, hat Secrets in die Logs geschrieben - GitHub-Tokens, Cloud-Credentials, was halt so im Runner-Speicher rumlag.

Workflows, die auf einen vollen Commit-SHA gepinnt waren, waren **nicht** betroffen. Punkt. Während die Tag-Nutzer ihre Secrets rotiert haben, haben die SHA-Nutzer Kaffee getrunken.

## Die Lösung: Pin auf die volle Commit-Hashes

Statt eines Tags (mutable) nutzt Du den 40 Zeichen langen Commit-SHA. Dieser ist unveränderlich - er zeigt für immer auf genau diesen einen Commit. Soll ein Angreifer sich daran versuchen, müsste er eine SHA-1-Kollision für ein gültiges Git-Objekt erzeugen. Das ist nicht das Bedrohungsmodell, das uns nachts wachhält.

```yaml
- uses: actions/checkout@692973e3d937129bcbf40652eb9f2f61becf3332 # v4.1.7
- uses: docker/build-push-action@v5         # ❌ mutable
- uses: hashicorp/setup-terraform@v3        # ❌ mutable
```

Den Versions-Tag als Kommentar dran lassen - dann sieht man auf einen Blick, welche Version eigentlich gemeint ist. Dependabot und Renovate können mit genau diesem Kommentar arbeiten und schicken Dir saubere PRs, wenn eine neue Version raus ist.

## Und warum jetzt? Weil GitHub das jetzt auch erzwingen kann

Seit August 2025 gibt es in den **Allowed-Actions-Policies** auf Org- und Enterprise-Ebene eine Checkbox, die SHA-Pinning **erzwingt**. Wer dann noch `@v4` schreibt, dessen Workflow scheitert beim Start - sauber, mit Fehlermeldung, ohne Diskussion. Wenn Du in einem Konzernumfeld arbeitest, ist es nur eine Frage der Zeit, bis die Security-Abteilung diesen Schalter umlegt. Besser, Deine Workflows sind dann schon vorbereitet.

Dazu kommt: Wer BSI-Grundschutz oder NIS2 ernst nimmt (siehe meine Posts dazu), kommt um die Frage "Wie stellst Du sicher, dass Deine CI/CD-Supply-Chain integer ist?" nicht herum. SHA-Pinning ist da kein Nice-to-have, sondern ein dokumentierbares Kontrollmittel.

## Ein Beispiel: Migration in der Praxis

Manuell ist das in jedem nicht-trivialen Repo eine Strafe. Drei Tools, die das übernehmen:

**1. `pinact`** - CLI-Tool, ersetzt Tags durch SHAs in einem Rutsch:

```bash
brew install suzuki-shunsuke/pinact/pinact
pinact run
# fertig - Workflow-Dateien sind gepinnt, Versions-Kommentare gesetzt
```

**2. Die Action `zgosalvez/github-actions-ensure-sha-pinned-actions`** - läuft als erster Job in Deinem CI und bricht ab, wenn jemand einen Tag committet:

```yaml
- uses: zgosalvez/github-actions-ensure-sha-pinned-actions@v3
  with:
    allowlist: |
      actions/
      aws-actions/
```

So bleiben Deine Workflows auch in Zukunft sauber, ohne dass Du in jedem Review danach gucken musst.

**3. Dependabot** - die `dependabot.yml`, die Du eh schon hast, kümmert sich auch um GitHub-Actions-Updates:

```yaml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

Damit bekommst Du wöchentlich PRs, die die SHAs auf den jeweils neuesten Tag-Stand heben - mit Diff, mit Changelog, mit Review.

## Fazit

Tags sind bequem, aber sie sind beweglich. SHAs sind unbequem, aber sie sind ehrlich. Nach `tj-actions` gibt es eigentlich keine vernünftige Ausrede mehr, in produktiven Workflows noch `@v4` stehen zu haben. Die Tools, die einem die Arbeit abnehmen, sind alle da. Der Aufwand für ein mittelgroßes Repo liegt bei einer halben Stunde - inklusive Kaffee.

Und falls Du das jetzt liest und denkst "machen wir nächstes Quartal": Genau das hat im März 2025 viele Teams in eine sehr lange Woche geschickt.

Wenn Du mehr erfahren möchtest, schau doch mal unter:
- [GitHub Docs: Security hardening for GitHub Actions](https://docs.github.com/en/actions/reference/security/secure-use)
- [GitHub Changelog: SHA pinning enforcement (Aug 2025)](https://github.blog/changelog/2025-08-15-github-actions-policy-now-supports-blocking-and-sha-pinning-actions/)