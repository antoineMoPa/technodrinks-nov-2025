---
theme: seriph
background: /lumen5.png
title: L'IA chez Lumen5
info: |
  ## L'IA chez Lumen5
  Architecture d'un service de production de vidéos assisté par l'IA
class: text-center
drawings:
  persist: false
transition: slide-left
mdc: true
---

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Host+Grotesk:ital,wght@0,300..800;1,300..800&display=swap" rel="stylesheet">

<h1 style="background: rgba(0,0,0,0.5); font-family: 'Host Grotesk', sans-serif;">Architecture d'un service de production de vidéos assisté par l'IA</h1>

---

# Lumen5?

- Plateforme lançée en 2017  par 3 co-fondateurs et amis à Vancouver
- Permet de convertir des articles de blogs en vidéo
- IA appliquée - On applique des technologies plutôt que de développer nos propres modèles
- Financement «bootstrap»
- Le marché: Marketing B2B
- Génère environ 4000 vidéos par jour avec l'IA

---

# Qui suis-je?

**Antoine Morin-Paulhus**

Développeur / Team lead dans l'équipe MARS.

MARS est une équipe full-stack qui s'occupe notamment de:

 - La connection avec les partenaires de média stock.
 - Plusieurs parties frontend de l'application.
 - L'IA générative.
 - Et plus encore!

Perso:

 - J'ai toujours des projets perso, présentement smoll.world (créez vos planètes en 3D)

---

# Petit historique de l'AI dans la plateforme Lumen5

- **2017 - 2022**
  - NLP
  - Compréhension simple et résumés de phrases
  <!-- l'IA et le traitement de language existait bien avant ChatGPT -->
- **2022**
  - ChatGPT apparaît
  <!-- 30 novembre 2022  -->
- **2023**
  <!-- Hackathon L5 -->
  - L'invasion des LLMs
  - Vidéos «talking head»
- **2024**
  - AI voiceover
- **2025**
  - Début de l'IA Générative dans la plateforme

---

# L'AI en prod partie 1

## Génération et raffinement de script de vidéo

<img src="/script_composer.png" class="m-auto max-h-96 mt-4" />

---

# Organisation des prompts dans le code

Nos prompts sont dans des fichiers .txt séparés, utilisant le language de template django:

```python
@inject_prompt_templates(
    [
        "script_generation/text_on_media_system.txt",
        "script_generation/voiceover_system.txt",
        "script_generation/user.txt",
    ]
)
def get_prompts_for_script_generation(
    # [...]
    text_on_media_system: str | None = None,
    voiceover_system: str | None = None,
    user: str | None = None,
    # [...]
) -> tuple[str, str]:
    """
    Returns the system and user prompts for generating a script!
    """
    # [...]
```

---

# User Prompt

Voici un extrait de notre prompt user:


```
Subject: {{ subject|safe }}
Audience: {{ audience|safe }}
Author: {{ author|safe }}

Reference material: {{ reference_material|safe }}

Other instructions: {{ other_instructions|safe }}
```

---

# System Prompt

Voici un extrait de notre prompt système:

```
You're a script writing assistant. Please write a script for a {{ script_type|safe }}
social media video for the provided title (and description, if provided).

First outline the planned sections with target {{ unit|safe }} for each,
adding to {{ target_description|safe }}. Then check if the {{ unit|safe }} count
add up to the target - if not, generate the planned sections again.
Then write the script.

Duration rules:
{% if script_type == "voiceover" %}
1. The script should be EXACTLY {{ target_words|safe }} words long.

[...]
```

C'est ici que notre magie opère! Nos prompts systèmes font en général 400-850 mots.

---

# Evals

On utilise des LLMs pour évaluer la performance de nos prompts par rapport à plusieurs exemples.

**Le concept:**

1. On crée un ensemble de cas de test avec des exemples variés
2. Un LLM évalue les résultats selon plusieurs critères (échelle 1-5):
   - **Style Transfer Accuracy** - Est-ce que le style correspond?
   - **Content Preservation** - Est-ce que le contenu est préservé?
   - **Content Grounding** - Y a-t-il des hallucinations?
3. On agrège les scores pour mesurer la performance globale

Cela nous permet d'itérer sur nos prompts de manière mesurable.

---

# L'AI en prod partie 2

## "Chatter" avec la plateforme

<!-- TODO: Add content -->

TODO

---

# L'AI en prod partie 3

## Recherche vectorielle: QDrant

Exemple: Trouver des images similaires en utilisant des embeddings d'image

<div class="grid grid-cols-2 gap-4 mt-4">
  <img src="/similar_images_1.png" class="rounded" />
  <img src="/similar_images_2.png" class="rounded" />
</div>

---

# Embedding

https://platform.openai.com/docs/guides/embeddings

**Cluster de vecteurs similaires**

<img src="/cluster_images_similaires.png" class="m-auto max-h-80 mt-4" />

---

## Recherche d'image similaires

**Comment ça fonctionne**

- On utilise l'API d'embedding d'OpenAI pour créer des vecteurs qui représentent chaque image à laquelle nous avons accès via Shutterstock
- On cherche les images dont les vecteurs sont les plus près dans notre DB Qdrant

**Pour la recherche par texte:**
- On convertit la requête en embedding et encore une fois, on cherche les vecteurs les plus près du vecteur de la requête

**Exemple live de qdrant:**
- https://food-discovery.qdrant.tech/

---

# L'AI en prod partie 4

## FAL.AI

**GIF:**
https://updates.lumen5.com/en/ai-generated-media

**LIVE DEMO** [FAL.AI](http://FAL.AI)
- Génération d'une image
- Modèle 3D

---

# L'AI en prod partie 5

## Voiceover avec Elevenlabs

<img src="/ai_voiceover.png" class="m-auto max-h-64 mt-4" />

```python
response = await self.async_client.text_to_speech.convert_with_timestamps(
    elevenlabs_voice.voice_id,
    text=text,
    model_id=model,
    voice_settings=elevenlabs_voice.settings,
    previous_text=join_word_list(previous_text, language),
    next_text=join_word_list(next_text, language),
    apply_text_normalization=(
        "on" if self.MODEL_SUPPORTS_TEXT_NORMALIZATION_ON.get(model, False) else "auto"
    ),
)
```

---

---

# Mini challenge

**Comment assurer un maximum de concurrence sans dépasser le rate-limit de ElevenLabs?**

**Problème:**<br/>
Les fournisseurs tels qu'ElevenLabs permettent seulement X requête par minute.

Comment maximiser notre efficacité sans dépasser les limites?

**Comment résoudre?**<br/>

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

# Question ouverte



## Comment rendre le système distribué?

Plusieurs serveurs chez Lumen5 peuvent contacter le même service.

---

# L'IA pour le développement

Bref portrait de l'IA dans l'équipe de Dev chez Lumen5 en 2025

**L'équipe a accès aux outils dev assistés par l'IA de son choix:**

- Claude
- Cursor
- Copilot

Les outils sont disponibles, l'adoption par les développeurs se fait de façon très variable et chacun y va à son rythme.

**Autres outils**

- Revue de code par un agent Claude
- Création de PR par un agent Claude à partir de Jira

<!-- TODO: Add images here -->

---
layout: center
class: text-center
---

# Questions?

Merci!
