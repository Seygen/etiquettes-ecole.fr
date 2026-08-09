# Ma classe — documents de rentrée

Application web gratuite pour les enseignantes et enseignants de maternelle :
saisir sa liste d'élèves une seule fois, puis imprimer tous les documents de rentrée.

- **Étiquettes** — présence, bannettes, porte-manteaux, en capitales / script / cursive
- **Pyramide des âges** — prénoms et dates de naissance, A4 paysage
- **Trombinoscope** — version classe, salle des maîtres ou cantine
- **Pages de garde** — une par élève, au thème de la classe
- **Groupes et suivi d'ateliers** — répartition automatique, affiche et feuille à cocher

## Confidentialité

**Aucune donnée ne quitte l'appareil.** Les prénoms, dates de naissance et photos
sont enregistrés uniquement dans le navigateur (`localStorage`). Il n'y a
aucun serveur, aucune base de données, aucun compte, aucun envoi sur internet.

L'import d'une liste ONDE au format PDF est lu **localement** dans le navigateur ;
les noms de famille sont écartés à la lecture, seuls le prénom, la date de
naissance et le sexe sont conservés.

Le bouton « Enregistrer ma liste dans un fichier » produit un fichier `.json`
téléchargé sur l'appareil. **Ce fichier contient des données d'élèves : il ne doit
jamais être committé dans ce dépôt** (le `.gitignore` l'exclut par précaution).

## Architecture

Site **100 % statique**, un seul fichier :

```
index.html          l'application entière (HTML + CSS + JavaScript en ligne)
.nojekyll           désactive tout traitement Jekyll côté GitHub Pages
.github/workflows/  déploiement automatique
```

Deux ressources sont chargées depuis internet à l'ouverture de la page :

| Ressource | Origine | Si indisponible |
|---|---|---|
| Polices (Baloo 2, Andika, Dancing Script, Nunito) | `fonts.googleapis.com` | polices système de remplacement, l'application reste utilisable |
| Lecteur PDF `pdf.js` 3.11.174 | `cdnjs.cloudflare.com` | seul l'import PDF est désactivé, avec un message explicite |

Aucune autre dépendance : pas de backend, pas d'API, pas de base de données,
pas de build, pas de `node_modules`.

## Déploiement

Le site est publié par GitHub Pages via GitHub Actions.
**Tout push sur `main` redéploie automatiquement** — il n'y a rien à faire d'autre.

Adresse de production : voir la section *Deployments* / *github-pages* du dépôt.

## Développement local

Aucun outil requis : ouvrir `index.html` dans un navigateur.

Pour tester dans des conditions identiques à GitHub Pages (servi depuis un
sous-chemin) :

```bash
python3 -m http.server 8000
# puis ouvrir http://localhost:8000/
```

## Nom de domaine personnalisé (à faire plus tard)

Le fichier `CNAME` n'est **volontairement pas présent** : il ne doit être créé
qu'une fois le domaine réellement acheté, avec le vrai nom de domaine dedans.
La procédure est décrite dans le fichier `DOMAINE.md`.

## Licence

Tous droits réservés.
