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
Jag lärde mig att container-säkerhet behöver byggas in från början, inte läggas på i efterhand.  
Att byta till en nyare och slim basimage gav stor skillnad i attackyta och antal sårbarheter.  
Jag såg också att dependency-versioner spelar stor roll, till exempel skillnaden mellan Flask 1.0.0 och 3.0.0.  
SBOM är viktig eftersom den ger en tydlig inventering av vad som faktiskt finns i imagen.  
Med en SBOM går det snabbare att avgöra om vi påverkas när en ny CVE publiceras.  
Det hjälper även compliance-arbete eftersom man kan visa spårbarhet i supply chain.  
Gatekeeper förändrar arbetssättet genom att flytta säkerhetskrav till policy-lagret i klustret.  
Istället för manuella code review-regler får vi automatiskt enforcement vid deploy.  
Det gör att teamet får snabb feedback och mer konsekventa säkerhetsnivåer i Kubernetes.
