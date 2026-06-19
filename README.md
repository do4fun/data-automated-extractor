# 🤖 Pipeline Universel de Veille Technologique Open-Data (Claude AI)

Ce dépôt héberge un pipeline d'ingénierie de données automatisé qui cartographie et centralise les jeux de données et API massives à travers le monde. L'objectif de cet inventaire universel est d'alimenter des applications d'IA en connaissances factuelles externes (via des pipelines RAG) afin d'éradiquer les hallucinations.

Le système est piloté par l'intelligence artificielle **Claude (Anthropic)** et s'exécute de manière 100 % autonome, sans serveurs dédiés ni fichiers intermédiaires stockés localement sur le disque.

## 📌 Fonctionnalités Clés

* **Scans Multi-Métiers Global** : Couverture systématique de plus de 12 secteurs (Finance, Santé, Juridique, Météo, Démographie, Logistique, Transports, Syndicats, etc.).
* **Zéro Hallucination & Format Strict** : Utilisation du mode *Tool Choice (Function Calling)* d'Anthropic pour contraindre l'IA à produire un JSON standardisé parfaitement conforme.
* **Dédoublonnage Incrémental en RAM** : Le script récupère la base existante, fusionne les nouvelles découvertes et met à jour les entrées existantes (via une clé unique basée sur l'URL source), évitant la perte d'historique.
* **Architecture Serverless** : Exécution automatique programmée via GitHub Actions.

---

## 🛠️ Schéma des Données (`data/veille_datasets.json`)

Le fichier produit est un flux JSON structuré enveloppé dans un objet racine `"datasets"`. Chaque entrée respecte la structure suivante :

```json
{
  "id": "UUID-v4-unique",
  "nom_ressource": "Nom exact du registre",
  "editeur_organisation": "Entité responsable de la gouvernance",
  "domaine_principal": "Catégorie normalisée (ex: 'Medical & Sante')",
  "protocoles_acces": ["API REST", "S3 Bucket", "GraphQL"],
  "formats_fichiers": ["JSON", "Parquet", "CSV"],
  "frequence_mise_a_jour": "Cadence (Temps réel, Quotidien, Mensuel...)",
  "score_anti_hallucination": 5, // Évaluation de la fiabilité humaine de 1 à 5
  "methode_integration_technique": "Description technique de la méthode de requêtage",
  "url_source_donnees": "https://...",
  "url_documentation_integration": "https://.../api-docs" // Lien direct vers le guide développeur
}
```

---

## 🚀 Activation & Configuration

Pour mettre en route ce pipeline autonome sur votre dépôt, suivez scrupuleusement ces deux étapes de configuration :

### 1. Ajout de la Clé API Anthropic
1. Sur GitHub, accédez à l'onglet **Settings** (Paramètres) de votre dépôt.
2. Dans le menu de gauche, cliquez sur **Secrets and variables** > **Actions**.
3. Cliquez sur le bouton **New repository secret**.
4. Configurez le secret comme suit :
   * **Name** : `ANTHROPIC_API_KEY`
   * **Secret** : *(Collez votre clé API Claude commençant par `sk-ant-...`)*

### 2. Autorisation d'Écriture du Workflow
Par défaut, les scripts GitHub Actions ont uniquement le droit de lire votre dépôt. Vous devez leur accorder le droit d'écrire pour insérer le fichier JSON de veille :
1. Restez dans l'onglet **Settings** > **Actions** > **General**.
2. Descendez tout en bas jusqu'à la section **Workflow permissions**.
3. Cochez l'option **Read and write permissions**.
4. Cliquez sur **Save**.

---

## 📅 Cycle d'Exécution & Maintenance

* **Automatique** : Le fichier `.github/workflows/veille_donnees_claude.yml` est configuré avec un déclencheur Cron (`0 6 * * 1`). Il se réveille de manière autonome **tous les lundis matin à 06h00 UTC**.
* **Manuel (Sur demande)** : Vous pouvez forcer le lancement de la veille à tout moment. Allez dans l'onglet **Actions** de votre dépôt, sélectionnez le workflow *Automatisme de Veille Technologique*, puis cliquez sur le bouton **Run workflow**.

### Suivi de l'activité
Il n'est pas nécessaire d'ouvrir le fichier JSON pour valider le bon fonctionnement. Le message de commit automatique sur GitHub affiche directement l'activité du dédoublonneur sous cette forme :
> 📝 *Feeds de veille automatisée (Claude) : 8 nouveaux, 3 MAJ*

---

## ⚠️ Résolution des Problèmes (Troubleshooting)

* **Le fichier JSON n'apparaît pas après le premier lancement ?** Vérifiez dans les logs de l'Action que l'étape *Workflow permissions* (Étape 2 de ce guide) a bien été validée, sinon l'API GitHub rejettera la mise à jour avec une erreur `403 Forbidden`.
* **Claude dépasse la limite de jetons ?** Le modèle utilisé est `claude-3-5-sonnet-latest` configuré avec un plafond de `max_tokens: 4096`. Si la base de données devient massive (plus de 500 entrées), ajustez la logique du script pour faire des requêtes partitionnées par domaine principal.
