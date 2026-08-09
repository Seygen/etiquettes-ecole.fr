# Brancher le domaine `etiquettes-ecole.fr`

Objectif : qu'un utilisateur tape `https://etiquettes-ecole.fr` et arrive
directement sur l'application, **sans jamais voir d'adresse `github.io`**.

C'est exactement ce que produit la configuration ci-dessous. Une fois le domaine
personnalisé actif, GitHub Pages redirige `seygen.github.io/etiquettes-ecole.fr/`
**vers** `etiquettes-ecole.fr` — la redirection va donc dans le bon sens :
l'adresse `github.io` disparaît de la barre d'adresse au lieu d'y apparaître.

---

## Ordre des opérations (important)

GitHub recommande de **créer les enregistrements DNS avant** de déclarer le
domaine personnalisé. Dans l'autre sens, le site devient injoignable pendant
toute la durée de propagation, et le certificat HTTPS ne peut pas être délivré.

```
1. Activer GitHub Pages                (Settings → Pages → Source: GitHub Actions)
2. Vérifier que le site répond          https://seygen.github.io/etiquettes-ecole.fr/
3. Acheter le domaine
4. Créer les enregistrements DNS        (tableau ci-dessous)
5. Attendre que dig renvoie les bonnes valeurs
6. Déclarer le domaine personnalisé     (Settings → Pages → Custom domain)
7. Attendre « Certificate: Active »
8. Cocher « Enforce HTTPS »             (= redirection HTTP → HTTPS)
```

Le fichier `CNAME` n'est **volontairement pas présent dans ce dépôt** tant que
l'étape 4 n'est pas faite : un `CNAME` posé trop tôt rend le site inaccessible.
Il sera créé à l'étape 6 (voir plus bas).

---

## Étape 4 — Enregistrements DNS à créer chez le registrar

`etiquettes-ecole.fr` est un domaine **apex** (sans sous-domaine). La norme DNS
interdit un CNAME sur un apex : il faut donc des enregistrements **A** et **AAAA**,
pas un CNAME. Le `www` est en revanche un CNAME.

| Type | Nom | Valeur | TTL |
|------|-----|--------|-----|
| A | `@` | `185.199.108.153` | 3600 |
| A | `@` | `185.199.109.153` | 3600 |
| A | `@` | `185.199.110.153` | 3600 |
| A | `@` | `185.199.111.153` | 3600 |
| AAAA | `@` | `2606:50c0:8000::153` | 3600 |
| AAAA | `@` | `2606:50c0:8001::153` | 3600 |
| AAAA | `@` | `2606:50c0:8002::153` | 3600 |
| AAAA | `@` | `2606:50c0:8003::153` | 3600 |
| CNAME | `www` | `seygen.github.io.` | 3600 |

### Précisions

- **Les quatre A sont à créer, pas un seul.** C'est ce qui assure la redondance
  de GitHub Pages. Idem pour les quatre AAAA (à omettre seulement si le
  registrar ne gère pas l'IPv6).
- `@` désigne le domaine racine. Certains registrars veulent un champ **vide**,
  d'autres `etiquettes-ecole.fr.` — les trois notations sont équivalentes.
- La valeur du CNAME `www` est bien `seygen.github.io.` — **sans** le nom du
  dépôt. Le point final compte chez certains registrars (OVH, Gandi).
- **Supprimer** tout A / AAAA / CNAME préexistant sur `@` et `www` : page de
  parking, redirection commerciale du registrar, etc. Ils créent des conflits.
- Ne **pas** utiliser une « redirection web » / « web forwarding » proposée par
  le registrar à la place de ces enregistrements : ce n'est pas du DNS, et le
  certificat HTTPS ne pourra pas être émis.
- **Cloudflare** : mettre le proxy en **DNS only** (nuage gris) le temps de la
  délivrance du certificat, sinon la validation échoue.

---

## Étape 6 — Déclarer le domaine personnalisé

Une seule des deux méthodes suffit ; elles produisent le même résultat.

**Méthode A — interface GitHub (recommandée)**

`Settings` → `Pages` → *Custom domain* → saisir `etiquettes-ecole.fr` → `Save`.
GitHub crée lui-même le fichier `CNAME` à la racine du dépôt et lance la
vérification DNS.

**Méthode B — depuis ce dépôt**

Créer un fichier nommé `CNAME` (sans extension) à la racine, contenant
exactement une ligne :

```
etiquettes-ecole.fr
```

puis committer et pousser sur `main`. Le workflow déploie la racine du dépôt,
le fichier se retrouve donc à la racine du site publié, ce qui est l'endroit
attendu par GitHub Pages.

> Ce fichier ne doit jamais être supprimé ensuite : sans lui, GitHub Pages perd
> le domaine personnalisé au déploiement suivant.

---

## Étapes 7 et 8 — HTTPS et redirection HTTP → HTTPS

- Le certificat TLS est **émis automatiquement et gratuitement** par GitHub
  (Let's Encrypt). Rien à acheter, rien à installer, rien à renouveler.
- Dans `Settings` → `Pages`, attendre l'affichage **« Certificate: Active »**.
  Cela prend de quelques minutes à ~24 h après la propagation DNS.
- Cocher alors **`Enforce HTTPS`**. C'est précisément la redirection
  automatique HTTP → HTTPS demandée : `http://etiquettes-ecole.fr` renverra un
  301 vers `https://etiquettes-ecole.fr`.
- La case reste **grisée** tant que le certificat n'est pas prêt : c'est normal,
  il n'y a rien à forcer, il faut simplement attendre.

---

## Vérifier que `https://etiquettes-ecole.fr` ouvre bien l'application

### 1. Le DNS pointe au bon endroit

```bash
dig +short etiquettes-ecole.fr A
# attendu : les quatre adresses 185.199.108.153 → 185.199.111.153

dig +short etiquettes-ecole.fr AAAA
# attendu : les quatre adresses 2606:50c0:800x::153

dig +short www.etiquettes-ecole.fr CNAME
# attendu : seygen.github.io.
```

Tant que ces commandes ne renvoient rien ou renvoient autre chose, la
propagation n'est pas terminée — inutile de passer à la suite.

### 2. Le site répond en HTTPS, sans redirection

```bash
curl -sSI https://etiquettes-ecole.fr | head -1
# attendu : HTTP/2 200      (et non 301, 302 ou 404)
```

Un `200` ici signifie que `index.html` est servi **directement** sur le domaine :
c'est l'objectif.

### 3. C'est bien l'application qui est servie

```bash
curl -sS https://etiquettes-ecole.fr | grep -o '<title>.*</title>'
# attendu : <title>Ma classe — documents de rentrée</title>
```

### 4. La redirection HTTP → HTTPS fonctionne

```bash
curl -sSI http://etiquettes-ecole.fr | grep -iE '^(HTTP|location)'
# attendu : HTTP/1.1 301 ... puis  location: https://etiquettes-ecole.fr/
```

### 5. L'ancienne adresse renvoie vers le domaine, et non l'inverse

```bash
curl -sSI https://seygen.github.io/etiquettes-ecole.fr/ | grep -iE '^(HTTP|location)'
# attendu : 301 ... location: https://etiquettes-ecole.fr/
```

C'est le point qui garantit qu'aucun utilisateur ne verra jamais d'URL
`github.io` : la redirection va de `github.io` **vers** le domaine.

### 6. Le certificat est valide

```bash
curl -sSI https://etiquettes-ecole.fr >/dev/null && echo "certificat OK"
```

Si le certificat était invalide, `curl` échouerait avec une erreur TLS.

### 7. Vérification finale dans un navigateur

Ouvrir `https://etiquettes-ecole.fr` en **navigation privée** (pour éviter tout
cache et tout `localStorage` existant) et vérifier :

- le cadenas est affiché, sans avertissement ;
- la barre d'adresse affiche `etiquettes-ecole.fr`, pas `github.io` ;
- l'écran d'accueil « Ma classe » s'affiche ;
- après « Commencer », l'ajout d'un élève de test fonctionne et les onglets
  Étiquettes / Pyramide / Trombinoscope se remplissent.

---

## Effet du changement d'adresse sur les données déjà saisies

L'application stocke tout dans le `localStorage`, qui est **cloisonné par
domaine**. Une classe saisie sur `seygen.github.io` ne réapparaîtra pas sur
`etiquettes-ecole.fr` : ce sont deux espaces de stockage distincts.

Sans conséquence si le domaine est branché **avant** la diffusion du lien aux
enseignantes. Si l'adresse `github.io` a déjà été partagée, leur indiquer
d'exporter leur liste (« Enregistrer ma liste dans un fichier ») depuis
l'ancienne adresse, puis de la recharger (« Charger ma liste ») sur la nouvelle.
