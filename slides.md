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
addons:
  - slidev-component-progress
---

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Host+Grotesk:ital,wght@0,300..800;1,300..800&display=swap" rel="stylesheet">

<h1 style="background: rgba(0,0,0,0.5); font-family: 'Host Grotesk', sans-serif;">Architecture d'un service de production de vidéos assisté par l'IA</h1>

---
layout: image-right
image: /cofounders.png
---

# Lumen5 - intro

- Plateforme lançée en 2017  par <br/>3 co-fondateurs et amis à Vancouver
- Financement «bootstrap»
- Fonctions principales:
  - Convertir des articles de blogs en vidéo.
  - Convertir des meetings ou webinaires en vidéos plus courts et plus dynamiques.
  - Le tout dans le style visuel du client.

---

# Notre marché

- Le marché&nbsp;:
Marketing B2B (Business to Business)
  - Vidéos de communications d'affaire pour les médias sociaux
  - Vidéos explicatifs
  - Relation avec les investisseurs

---

# Technologies

- IA appliquée&nbsp;: On applique des technologies plutôt que de développer nos propres modèles
- Engin vidéo: Luminary (utilise PIXI.js)
- Django
- Celery
- React
- Nos clients créent environ 4000 vidéos par jour!


---

# Qui suis-je?

**Antoine Morin-Paulhus**

Développeur / Team lead dans l'équipe MARS.

MARS est une équipe full-stack qui s'occupe notamment de:

 - La connection avec les partenaires de média stock.
 - Plusieurs parties frontend de l'application.
 - L'IA générative.
 - Et plus encore!

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

# L'IA appliquée en prod - Cas 1<br/><span class="font-bold">Génération de script</span>

Application des LLMs pour la génération et le raffinement de script de vidéo

<img src="/script_composer.png" class="m-auto max-h-96 mt-4" style="width: 600px"/>

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

# Organisation des prompts dans le code

Pour mettre le tout ensemble:

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

# L'IA appliquée en prod - Cas 2<br/> <span class="font-bold">Recherche vectorielle</span>

Recherche vectorielle avec QDrant

Notre bibliothèque d'images permet de trouver des images similaires à une autre:

<img src="/similar_images_2.png" class="rounded m-auto" width="200px"/>

---

## Recherche d'image similaires

**Comment ça fonctionne**

- On trouve ou crée des représentation textuelles d'image.
- On utilise l'API d'embedding d'OpenAI pour créer des vecteurs qui représentent chaque image de nos collections.
- On cherche les images dont les vecteurs sont les plus près dans notre DB Qdrant

**Pour la recherche par texte:**
- On convertit la requête en embedding et encore une fois, on cherche les vecteurs les plus près du vecteur de la requête

**Exemple live de qdrant:**
- https://food-discovery.qdrant.tech/

---

# Embedding

https://platform.openai.com/docs/guides/embeddings

**Cluster de vecteurs similaires**

<img src="/cluster_images_similaires.png" class="m-auto max-h-80 mt-4" />

---

## Recherche d'image similaires

**Requête d'embedding via OpenAI**


```python
from openai import OpenAI
client = OpenAI()

response = client.embeddings.create(
    input="Your text string goes here",
    model="text-embedding-3-small"
)

print(response.data[0].embedding)
```

Exemple tiré de [https://platform.openai.com/docs/guides/embeddings?lang=python](https://platform.openai.com/docs/guides/embeddings?lang=python)


---

## Recherche d'image similaires

**Résultat d'embedding via OpenAI**


```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "index": 0,
      "embedding": [
        -0.006929283495992422,
        -0.005336422007530928,
        -4.547132266452536e-05,
        -0.024047505110502243
      ],
    }
  ],
  "model": "text-embedding-3-small",
  "usage": {
    "prompt_tokens": 5,
    "total_tokens": 5
  }
}
```

---


## Recherche d'image similaires

**Pistes d'amélioration**

* Utiliser un vecteur d'embedding utilisé directement à partir d'une image.
* Non supporté par OpenAI pour le moment.
* Possibilité d'utiliser un modèle de vision tel que ConvNeXt


---


# L'AI appliquée en prod - Cas 3<br/> <span class="font-bold">IA Générative</span>

Les clients exigent de plus en plus d'avoir accès à des outils d'IA générative.

<img src="/generative_images.gif" class="m-auto max-h-96 mt-4" />


---


# L'AI appliquée en prod - Cas 3<br/> <span class="font-bold">IA Générative</span>

Démo

**LIVE DEMO** [FAL.AI](http://FAL.AI)
- Génération d'une image
- Modèles Image -> video
- Génération de modèles 3D

---


# Architecture des services

Comment tout ceci est organisé?

<img src="/architecture.png" class="m-auto max-h-96 mt-4" />

---

# Sécurité et utilisation responsable de l'IA

Comment encadrer l'utilisation de l'IA?

**Les outils**

* API de modérations des fournisseurs:
  * FAL.AI rend très difficile la génération de contenu NSFW
  * OpenAI donne accès à un API de modération.
* Fonctionalités de vérification
  * Groundhog
    * Un outil supplémentaire pour valider l'information véhiculée dans nos scripts.

**Le choix du marché**

  * Les communicateurs B2B utilisant notre plateforme créent du contenu en général plus professionel.


---

# Exemple de modération OpenAI

Exemple de classification de texte

```python
from openai import OpenAI
client = OpenAI()

response = client.moderations.create(
    model="omni-moderation-latest",
    input="...text to classify goes here...",
)

print(response)
```

[https://platform.openai.com/docs/guides/moderation](https://platform.openai.com/docs/guides/moderation)


---

# Exemple de modération FAL.ai

FAL.AI fait sa propre modération


<img src="/fal_ai_safety.png" class="m-auto max-h-96 mt-16" width="700px"/>


---

# L'AI appliquée en prod - Cas 4<br/> <span class="font-bold">Des voix artificielles</span>

Voiceover avec Elevenlabs

Dans lumen5:

<img src="/ai_voiceover.png" class="m-auto max-h-64 mt-4" width="600px"/>

- Elevenlabs:
  - <span class="font-bold">Input</span>: du texte écrit
  - <span class="font-bold">Output</span>: un fichier audio avec une voix artificielle



---

# L'AI appliquée en prod - Cas 4<br/> <span class="font-bold">Des voix artificielles</span>

Les voix artificielles, opportunités et risques:

- Vidéos lues par l'IA (Lumen5 et autres)
- Fraude par clone de la voix
- Donner la voix à ceux qui n'en ont pas ou plus

À écouter:

[Ces secrets que nos voix livrent à l’IA](https://ici.radio-canada.ca/ohdio/premiere/emissions/tout-terrain/segments/rattrapage/1959945/mariage-voix-et-ia-reportage-yanik-dumont-baronhttps://ici.radio-canada.ca/ohdio/premiere/emissions/tout-terrain/segments/rattrapage/1959945/mariage-voix-et-ia-reportage-yanik-dumont-baron) 🔊

---


# Questions?

Merci!
