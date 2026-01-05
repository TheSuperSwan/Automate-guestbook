# Automate-guestbook
# Guestbook – CI/CD med GitHub Actions, GHCR och OpenShift

Detta projekt bygger vidare på Guestbook-applikationen och syftar till att demonstrera en komplett CI/CD-pipeline med GitHub Actions, GitHub Container Registry (GHCR) och OpenShift. Fokus i uppgiften ligger inte på att utveckla en ny applikation, utan på att automatisera bygg, publicering och deployment av containers.

---

## Grupp

**Gruppnamn:** Grupp 5  
**Deltagare:**
- Baran Kizilca  
- Viktor Svanholm
- Gabriella Öhrn

---

## Applikation och arkitektur

Applikationen består av tre containers:

- **Backend** – Golang-baserat API  
- **Frontend** – Nginx som serverar HTML/JavaScript och proxyar API-anrop
- **Databas** - Postgresql databas

---

## CI – Build och publicering av containers

Vid varje push till `main` triggas en GitHub Actions-workflow som automatiskt:

- Checkar ut koden från repot  
- Bygger backend-containern från `guestbook-backend/Dockerfile`  
- Bygger frontend-containern från `guestbook-frontend/Dockerfile`  
- Publicerar båda images till GitHub Container Registry (GHCR)  

Exempel på images som skapas:

- `ghcr.io/thesuperswan/guestbook-backend:latest`  
- `ghcr.io/thesuperswan/guestbook-frontend:latest`

Postgresql-imagen är den enda som inte ligger på GHCR, eftersom den inte behövde byggas om.

---

## CD – Deployment till OpenShift

Samma GitHub Actions-workflow ansvarar även för deployment till OpenShift.

Deploymenten sker genom att workflowet:

- Installerar OpenShift CLI (`oc`)  
- Loggar in mot OpenShift  
- Väljer rätt projekt/namespace  
- Kör `oc apply -f openshift/`  

Alla OpenShift-resurser (Deployments, Services, Routes m.m.) ligger samlade i mappen `openshift/`.

---

## Autentisering mot OpenShift

I ett CI/CD-sammanhang fungerar inte alltid användarnamn och lösenord eller kortlivade webbtokens. Därför används en **ServiceAccount i OpenShift** för automatiserad deployment.

Flödet är följande:

- En ServiceAccount skapas i OpenShift  
- ServiceAccounten tilldelas rättigheter i projektet  
- Dess token lagras som en GitHub Secret  
- GitHub Actions använder denna token för att autentisera mot OpenShift  

Detta är standardpraxis i Kubernetes/OpenShift och ger en stabil och säker lösning för automation.

---

## Secrets i GitHub

Följande secrets används i GitHub Actions:

- `OCP_SERVER` – OpenShift API-server  
- `OCP_NAMESPACE` – OpenShift-projekt/namespace   

Inga känsliga uppgifter är hårdkodade i repot.

---

## Bonus – Resonemang

### Vilka YAML-filer ändras vid uppdateringar?
Vanligtvis är det **Deployment-filerna** som ändras när en ny image-version ska användas. Services och Routes behöver sällan ändras.

### Behöver man köra `oc apply` på alla filer?
Nej. Det räcker att köra `oc apply` på de YAML-filer som har ändrats. I detta projekt används dock `oc apply -f openshift/` för enkelhetens skull.

### Vad händer om en ConfigMap ändras?
Om en ConfigMap ändras behöver tillhörande Deployment startas om (rollout) för att ändringarna ska slå igenom.

---

## Sammanfattning

Projektet visar ett komplett CI/CD-flöde där:

- GitHub Actions automatiskt bygger containers  
- Images publiceras till GHCR  
- Applikationen deployas till OpenShift utan manuell inblandning  
- Säker autentisering hanteras via Secrets och ServiceAccount  

Detta uppfyller samtliga krav i uppgiften och demonstrerar hur en modern, automatiserad containerbaserad deployment kan byggas.
