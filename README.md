# Lab 2 - Container Security

## Översikt
Detta repo visar en sårbar container (`Dockerfile.vulnerable`) och en härdad container (`Dockerfile.hardened`), inklusive Trivy-skanning före/efter, SBOM i CycloneDX-format och Gatekeeper-policyfiler.

## Verktyg
- Docker
- Trivy
- OPA Gatekeeper (via Mission Control -> Lab Hub)

## Filer
- `Dockerfile.vulnerable`: medvetet sårbar image (`python:3.8`, `flask==1.0.0`)
- `Dockerfile.hardened`: härdad image (`python:3.12-slim-bookworm`, non-root user, healthcheck)
- `scan-before.txt`: Trivy-resultat för sårbar image
- `scan-after.txt`: Trivy-resultat för härdad image
- `sbom.json`: CycloneDX SBOM för härdad image
- `policies/*.yaml`: Gatekeeper policy template + constraint

## Vad som förändrades
- `python:3.8` -> `python:3.12-slim-bookworm`
- `flask==1.0.0` -> `flask==3.0.0`
- Non-root user (`appuser`)
- `pip --no-cache-dir`
- Healthcheck tillagd

## Trivy-resultat (CRITICAL/HIGH)
- Före: `Total: 1440 (HIGH: 1259, CRITICAL: 181)`
- Efter: `Total: 4 (HIGH: 2, CRITICAL: 2)`
- Notering: De kvarvarande 4 HIGH/CRITICAL-fynden ligger i OS-baspaket (Debian Bookworm i `python:3.12-slim-bookworm`), inte i applikationsdependencies.

## Screenshots
- `trivy-before.png` visar scan av sårbar image (`my-app:vulnerable`) med hög mängd HIGH/CRITICAL findings.
  ![Trivy Before](screenshots/trivy-before.png)
- `trivy-after.png` visar scan av härdad image (`my-app:hardened`) med kraftigt reducerat antal HIGH/CRITICAL findings.
  ![Trivy After](screenshots/trivy-after.png)
- `gatekeeper-deny.png` visar ett avslag (DENIED by Gatekeeper) för en osäker pod som bryter mot flera policies.
  ![Gatekeeper Deny](screenshots/gatekeeper-deny.png)
- `gatekeeper-pass.png` visar en tillåten pod (ALLOWED) i dry-run med varning om default service account.
  ![Gatekeeper Pass](screenshots/gatekeeper-pass.png)

## Gatekeeper-status
Gatekeeper Lab i Mission Control användes för att deploya policies och köra dry-run tester.  
`Bad Pod` gav `DENIED by Gatekeeper`, vilket visar att policy-enforcement blockerar osäkra pods.  
`Hardened Pod` gav `ALLOWED with warnings`, vilket visar att podden tillåts men fortfarande får rekommendationer (t.ex. undvika default service account).

## Reflektion
Jag lärde mig att en uppdaterad och mindre image med patchade komponenter minskar risken för sårbarheter.  
Jag såg också att SBOM fungerar som en inventeringslista över alla komponenter i imagen, vilket gör det enklare att analysera CVE:er och felsöka snabbare.  
Gatekeeper förändrar arbetssättet från manuella kontroller till automatiska policyregler i Kubernetes.  
Det gör att osäkra resurser kan stoppas direkt vid deploy i stället för att upptäckas i efterhand.
