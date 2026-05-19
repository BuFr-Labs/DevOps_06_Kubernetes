# DevOps_06_Kubernetes
Repozitoř k 6. lekci

# DevOps Úkol 06: Práce s Kubernetes – Deploying Nginx pomocí Minikube

Tento repozitář obsahuje řešení domácího úkolu zaměřeného na seznámení se se základy Kubernetes, deklarativní nasazení webového serveru Nginx a konfiguraci síťových služeb v lokálním vývojovém clusteru Minikube.

Celé řešení bylo prakticky realizováno a otestováno v prostředí operačního systému Red Hat Enterprise Linux 10 (RHEL 10).

## 🎯 Cíle projektu
* Inicializace lokálního jednodevítkového Kubernetes clusteru pomocí nástroje Minikube.
* Deklarativní definice a nasazení aplikace (Nginx) s požadavkem na vysokou dostupnost (2 repliky) pomocí Deploymentu.
* Vytvoření síťové abstrakce typu Service (NodePort) pro stabilní směrování provozu a vystavení aplikace na specifickém portu 30007.
* Ověření konceptu horizontálního škálování (scale up ze 2 na 3 instance) a sledování reakce clusteru v reálném čase.
* Správné síťové propojení a zpřístupnění aplikace ze vzdáleného hostitelského počítače přes SSH tunel a úpravu lokálního firewallu.

## 📂 Struktura projektu
Projekt je rozdělen do čistých deklarativních konfiguračních YAML souborů a dokumentace:

* `nginx-deployment.yaml` - Definice Deploymentu, která k8s říká, jaký image (Nginx) má stáhnout, kolik instancí (replicas: 2) má neustále udržovat v běžícím stavu a na jakém vnitřním portu (80) aplikace naslouchá.
* `nginx-service.yaml` - Definice síťové služby typu NodePort, která propojuje vnější port uzlu (30007) s vnitřním portem Service a přes selektory dynamicky balancuje provoz na žijící pody.
* `kompletni_protokol_kubernetes_v2.pdf` - Detailní a strukturovaný protokol vygenerovaný přímo z logů terminálu zachycující stavy, eventy, aplikační logy a test elasticity.

## 🚀 Návod k použití

### 1. Prerekvizity
* Nainstalovaný a spuštěný **Docker** na cílovém Linux systému.
* Uživatelská oprávnění pro spuštění Dockeru bez sudo (`usermod -aG docker $USER`).
* Stažené binární soubory **Minikube** a **kubectl** umístěné v systémové cestě.

### 2. Spuštění a inicializace clusteru
Nastartování lokálního Kubernetes clusteru:
```bash
minikube start
```
Ověření stavu uzlu a získání základních informací o běžícím clusteru:

```Bash
kubectl cluster-info
kubectl get nodes
```

### 3. Nasazení aplikace a síťové služby
Aplikace deklarativního Deploymentu (vytvoření podů s Nginx):

```Bash
kubectl apply -f nginx-deployment.yaml
```

Aplikace síťové služby NodePort (vystavení portu 30007):

```Bash
kubectl apply -f nginx-service.yaml
```

Kontrola, zda jsou všechny komponenty správně vytvořeny a pody jsou ve stavu Running:

```Bash
kubectl get all
```

### 4. Zpřístupnění a testování v lokální síti
V prostředí vzdálené virtuálky bez grafického rozhraní je nutné provoz přesměrovat na hostitelský stroj. Spuštění port-forwardu na všechna rozhraní:

```Bash
kubectl port-forward --address 0.0.0.0 service/my-nginx 8080:80
```

Na hostitelském systému (např. Windows) pak stačí v prohlížeči otevřít adresu s IP adresou vaší virtuálky:

```Plaintext
http://<IP_TVOJI_VIRTUALKY>:8080
```

### 5. Test horizontálního škálování (Elasticita)
Zvýšení počtu běžících instancí Nginxu za provozu z původních 2 na 3 repliky:

```Bash
kubectl scale deployment my-nginx --replicas=3
```

Okamžité ověření, že Kubernetes bleskově nastartoval nový pod a deployment vykazuje stav 3/3:

```Bash
kubectl get pods
kubectl get deployments
```

### 6. Úklid clusteru (Cleanup)
Pro uvolnění systémových prostředků a smazání všech vytvořených objektů z clusteru:

```Bash
kubectl delete -f nginx-service.yaml
kubectl delete -f nginx-deployment.yaml
```

### Monitorování a debugging
V průběhu nasazení byly pro ověření interních mechanismů Kubernetes využívány následující diagnostické příkazy:

* ```bash kubectl logs -l run=my-nginx``` - Sledování přístupových logů (včetně úspěšných HTTP kódů 200 z prohlížeče).
* ```bash kubectl get events --sort-by=.metadata.creationTimestamp``` - Chronologický výpis událostí (stahování image, přiřazení podů uzlům).
* ```bash kubectl describe pod <nazev-podu>``` - Detailní inspekce životního cyklu konkrétního kontejneru.