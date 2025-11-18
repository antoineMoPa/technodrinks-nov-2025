# Mini challenge

**Comment assurer un maximum de concurrence sans dépasser le rate-limit de ElevenLabs?**

**Problème:**<br/>
Les fournisseurs tels qu'ElevenLabs permettent seulement X requête par minute.

Comment maximiser notre efficacité sans dépasser les limites?

**Comment résoudre?**<br/>

Réponse à la prochaine page 👀

---

# Mini challenge

**Comment assurer un maximum de concurrence sans dépasser le rate-limit de ElevenLabs?**

**Exemple de solution:**
- Utiliser un sémaphore `asyncio.Semaphore`

```python
import asyncio

async def worker(semaphore, id):
    async with semaphore:
        print(f"Worker {id} acquired semaphore")
        await asyncio.sleep(1)  # Simulate some work
        print(f"Worker {id} released semaphore")

async def main():
    semaphore = asyncio.Semaphore(2)  # Allow 2 concurrent workers
    tasks = [worker(semaphore, i) for i in range(5)]
    await asyncio.gather(*tasks)

if __name__ == "__main__":
    asyncio.run(main())
```

---

# Output

```
python3.11 semaphores.py
Worker 0 acquired semaphore
Worker 1 acquired semaphore
Worker 0 released semaphore
Worker 1 released semaphore
Worker 2 acquired semaphore
Worker 3 acquired semaphore
Worker 2 released semaphore
Worker 3 released semaphore
Worker 4 acquired semaphore
Worker 4 released semaphore
```

---



---
layout: image-right
image: /old-man-yells-at-cloud.png
---

# Hébergement

## Gestion du cluster

* GCP (Google Cloud Platform)
* On adopte une approche <i>Infrastructure as Code</i>
  * Terraform
  * Flux

## Monitoring

* Prometheus
* AlertManager
* Sentry


---
layout: image-right
image: /claude.png
---

# L'IA pour le développement

Nos outils

**L'équipe a accès aux outils dev assistés par l'IA de son choix:**

- Claude
- Cursor
- Copilot

**Autres outils**

- Revue de code par un agent Claude
- Création de PR par un agent Claude à partir de Jira

---
