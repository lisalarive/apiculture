# 🐝 API-Culture : Quiz de Culture Générale Interactif

**Lien vers le site :** [API-Culture](https://larivelisa.dev/apiculture/frontend/index.html)

Application web interactive de quiz, développée dans le cadre de l'UE **LU2IN013 (Projet de développement)** en Licence d'Informatique à **Sorbonne Université** (Année 2025-2026).

🏆 **Note obtenue : 18,6/20**

## 🎯 Concept et Chiffres Clés

API-Culture a pour ambition de transformer le temps d'écran passif en un moment d'apprentissage ludique. Le quiz se déroule en sessions de 10 questions réparties sur **6 thématiques** (Sciences & Ingénierie, Musique, Cinéma, Histoire & Littérature, Nature & Géographie, Internet).

*   **+ de 27 000 questions** uniques générées par IA.
*   **+ de 1 200 pages Wikipédia** utilisées comme source de données.
*   **Répartition des difficultés :** Facile (50%), Moyen (35%), Difficile (15%).

## ⚙️ Architecture et Stack Technique

L'application repose sur une architecture découplée, séparant clairement la génération des données (offline) du jeu en lui-même (online).

*   **Backend :** Python 3, FastAPI, SQLite3
*   **Frontend :** HTML5, CSS3, JavaScript (Vanilla)
*   **API Tierces :** Wikimedia API, Gemini API (`gemini-3.1-flash-lite-preview`)
*   **Infrastructure & Qualité :** Docker, Pipeline CI/CD GitLab, Tests unitaires et fonctionnels

## 🗄️ Modélisation de la Base de Données (SQLite)

Pour contourner les limites de requêtes des API en temps réel, l'ensemble des questions a été pré-généré et stocké dans une base relationnelle SQLite structurée autour de 3 entités principales :
1.  **Theme :** (`idt`, `nom`)
2.  **Page_wiki :** (`idpw`, `titre_page`, `est_traitee`, `#idt`) - Permet le suivi des pages déjà explorées.
3.  **Reponse_gemini :** (`idrg`, `question`, `reponse`, `difficulte`, `date_ajout`, `#idpw`) - Stocke les données générées.

## 🧠 Génération de Données (Pipeline Python)

Le remplissage de la base de données s'effectue via un script Python automatisé :
1.  **Extraction (API Wikimedia) :** Récupération du texte brut des articles (via les paramètres `prop=extracts` et `explaintext=1` pour exclure le HTML).
2.  **Calcul de volume :** Génération d'environ 1 question tous les 250 mots pour équilibrer le traitement des articles courts et longs.
3.  **Prompt Engineering (API Gemini) :** Envoi du texte au LLM avec des consignes strictes (ex: obligation de citer le sujet dans la question, inclusion d'au moins une question "devinette").
4.  **Formatage et Fiabilité :** Forçage de la sortie au format JSON et validation via des schémas **Pydantic** (`QuizOutput`, `QuizQuestion`). La température du modèle est fixée à `0.2` pour limiter les hallucinations.

## 🚀 Logique Métier (FastAPI)

Le lien entre la base SQLite et le Frontend est assuré par deux routes FastAPI :

*   **Génération d'une session (`/question/{theme}`) :** Implémentation d'un **double tirage au sort** (regroupement par page Wikipédia pour n'en garder qu'une, puis mélange final) garantissant au joueur 10 questions issues de 10 sujets différents.
*   **Correction intelligente (`/verifier_reponse_joueur`) :**
    *   *Normalisation :* Mise en minuscules, suppression des accents, espaces et articles (le, la, les...).
    *   *Distance de Levenshtein :* Tolérance adaptative aux fautes de frappe selon la longueur du mot attendu (0 faute pour < 4 lettres, 1 faute entre 5 et 10 lettres, 2 fautes pour > 10 lettres).

## 🎨 Frontend, UX et State Management

*   **Maquettage & Tests Utilisateurs :** Conception sur **Figma** et validation via des tests d'utilisabilité. Les retours ont conduit à des ajustements ergonomiques (ajout de flèches de retour, menus plus explicites).
*   **Gestion d'état (`localStorage`) :** L'intégralité de l'état de la partie (questions pré-chargées, score, numéro de question) et les statistiques globales du joueur (parties jouées, score moyen, etc.) sont gérés côté client.
*   **Auto-correction ("J'avais la bonne réponse") :** Fonctionnalité ajoutée pour pallier la frustration d'un correcteur algorithmique parfois trop strict, permettant au joueur de s'auto-attribuer un point basé sur la confiance.
*   **Responsive Design & UI :** Charte graphique autour du monde des abeilles (polices Kavoon / Meera Inimai), effets de survol marqués, et conception *Mobile First* (suppression des éléments superflus sur petit écran, réorganisation des grilles CSS).

## 🏗️ Déploiement et Tests

*   **Conteneurisation :** L'application est packagée avec **Docker**.
*   **CI/CD :** Pipeline configurée sur GitLab (`.gitlab-ci.yml`) pour automatiser le build à chaque push.
*   **Qualité logicielle :** Couverture du code par des tests unitaires (tests de l'algorithme de Levenshtein, de la normalisation) et validation complète via des scénarios de tests fonctionnels documentés.
*   **Easter Egg :** Exploitation d'une faille de navigation via le bouton "retour" permettant de dépasser le score maximal, transformée en *easter egg* (message spécial pour un score > 10).

---
*Projet réalisé par Lisa Larive et Eric Li.*
