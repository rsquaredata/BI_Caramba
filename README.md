# Projet BI - Caramba (Entrepôt de données + Tableau)
Master 2 SISE - Université Lyon 2 (2025)

## Objectif du projet
Ce projet vise à construire un **entrepôt de données décisionnel** appliqué aux ventes de bonbons de l'entreprise Caramba, modélisé en schéma **snowflake** dans Oracle, enrichi sur trois axes, et exploité dans Tableau selone une approche **HOLAP** (Hybrid OLAP : stockage relationnel + analyse multidimensionnelle).

Le projet met en œuvre :
- modélisation décisionnelle (table de faits, dimensions, hiérarchie),
- enrichissements **analytique, géographique et métier**,
- implémentation SQL dans Oracle,
- préparation des données,
- création d'un dashboard Tableau pour la navigation OLAP.

---
## Structure du dépôt
```
bi_caramba/
├── README.md
├── data/
│   ├── raw/             
│   │   ├── f_vente_bonbon.csv
│   │   └── d_*.csv
│   └── final/           
│       └── entrepot.xlxs
│
├── sql/
│   ├── schema_caramba_snowflake.sql
│   └── inserts_exemple.sql
│
├── tableau/
│   ├── caramba_dashboard.twbx
│   └── screenshots/
│       ├── vue_kpi.png
│       ├── vue_geo.png
│       └── vue_evenements.png
│
└── docs/
    ├── rapport_caramba.pdf
    ├── rapport_caramba.tex
    └── annexe_tableau/
        ├── screenshot1.png
        ├── screenshot2.png
        └── ...
```
---
## Enrichissements intégrés
1. Enrichissement analytique : indice ventes nationales du secteur de la confiserie (Insee)
2. Enrichissement géographique : population, revenus médian, région
3. Enrichisement métier : événements
---
## Modélisation : schéma snowflake et approche HOLAP
Le modèle décisionnel repose sur :
- une table de faits (vente)
- des dimensions normalisées : Temps, Magasin, Géographie, Événements
- une logique multidimensionnelle exploitée dans Tableau
---
## Implémentation Oracle et requêtes OLAP
1.Création du schéma
À àpartir du script `schema_caramba_snowflake.sql` :
- création des dimensions et de la table de faits
- définition des clés primaires/étrangères
- vérification de l'intégrité référentielle
2. Alimentation
- données initiales (`/data/raw/`)
- enrichissements (`/data/enrich/`)
- tables finales (`/data/final/`)
- insertion dans oracle via `inserts_exemple.sql` (version démo)`
- 3. Requêtes OLAP relationnelles
Roll-up : CA annuel
```sql
SELECT 
    t.annee,
    SUM(f.chiffre_affaire) AS ca_annuel
FROM vente_bonbon f
JOIN periode p 
    ON f.mois = p.mois
JOIN trimestriel t 
    ON p.trimestre = t.trimestre
GROUP BY t.annee
ORDER BY t.annee;
```
Drill-down : CA trimestriel pour une année donnée
```sql
SELECT 
    t.trimestre,
    SUM(f.chiffre_affaire) AS ca_trimestriel
FROM vente_bonbon f
JOIN periode p 
    ON f.mois = p.mois
JOIN trimestriel t 
    ON p.trimestre = t.trimestre
WHERE t.annee = 2014
GROUP BY t.trimestre
ORDER BY t.trimestre;
```
Slice : Analyse des ventes pendant les événements
```sql
SELECT 
    e.evenement,
    SUM(f.chiffre_affaire) AS ca_evenement
FROM vente_bonbon f
JOIN evenements e 
    ON f.mois_normalise = e.mois
GROUP BY e.evenement
ORDER BY ca_evenement DESC;
```
Dice : Analyse événementielle par type de bonbon
```sql
SELECT 
    e.evenement,
    b.nom AS type_bonbon,
    SUM(f.chiffre_affaire) AS ca_total
FROM vente_bonbon f
JOIN evenements e 
    ON f.mois_normalise = e.mois
JOIN type_bonbon b 
    ON f.nom = b.nom
GROUP BY e.evenement, b.nom
ORDER BY e.evenement, ca_total DESC;
```
4. Exploitation Tableau
Les données Oracle ont été exploitées dans Tableau pour :
- naviguer dans les hiérarchies (Temps, Magasin, Géographie),
- effectuer les opérations OLAP (roll-up, drill-down, slice, dice),
- produire des dashboards analytiques.
---
## Points forts du projet
- schéma snowflage propre et cohérent
- enrichissements intégrés
- implémentation Oracle selon un schéma logique + requêtes OLAP
- dashboards Tableau riches et interactifs
- rapport structuré et argumenté
- travail reproductible
---
## Axes d'amélioration
- meilleure intégration des enrichissements (moins d'étapes manuelles)
  passage à un modèle en constellation avec plusieurs tables de faits
- pipeline ETL automatisé
- actualisation automatique dans Tableau
---
## Crédits
Projet réalisé dans le cadre du M2 SISE avec :
- Abdourahmane TIMERA
- Cyrille PECNIK
- Perrine IBOUROI
- Rina RAZAFIMAHEFA
