# Brancher le nom de domaine `etiquettes-ecole.fr`

> ⚠️ **Rien à faire tant que le domaine n'est pas acheté.**
> Le fichier `CNAME` n'existe pas encore : c'est normal et voulu.
> Un `CNAME` contenant un domaine non acheté ferait tomber le site en 404.

Une fois le domaine acheté (OVH, Gandi, Infomaniak, Cloudflare…), la mise en
route se fait en trois temps, dans cet ordre.

---

## 1. Chez le registrar : créer les enregistrements DNS

### Pour le domaine racine `etiquettes-ecole.fr`

Créer **quatre enregistrements A** (les quatre, pas un seul — c'est ce qui assure
la redondance) :

| Type | Nom / Hôte | Valeur | TTL |
|---|---|---|---|
| A | `@` | `185.199.108.153` | 3600 |
| A | `@` | `185.199.109.153` | 3600 |
| A | `@` | `185.199.110.153` | 3600 |
| A | `@` | `185.199.111.153` | 3600 |

Et, si le registrar gère l'IPv6, **quatre enregistrements AAAA** :

| Type | Nom / Hôte | Valeur | TTL |
|---|---|---|---|
| AAAA | `@` | `2606:50c0:8000::153` | 3600 |
| AAAA | `@` | `2606:50c0:8001::153` | 3600 |
| AAAA | `@` | `2606:50c0:8002::153` | 3600 |
| AAAA | `@` | `2606:50c0:8003::153` | 3600 |

### Pour `www.etiquettes-ecole.fr`

Un seul enregistrement **CNAME** :

| Type | Nom / Hôte | Valeur | TTL |
|---|---|---|---|
| CNAME | `www` | `seygen.github.io.` | 3600 |

> Le point final après `seygen.github.io.` est important chez certains registrars.
> Ne pas mettre le nom du dépôt dans cette valeur : uniquement `seygen.github.io.`

### Points d'attention

- **Supprimer** tout enregistrement A, AAAA ou CNAME préexistant sur `@` et `www`
  (page de parking du registrar, redirection web, etc.) : ils entreraient en conflit.
- Ne **pas** utiliser une « redirection web » ou un « web forwarding » proposé par
  le registrar : ce n'est pas la même chose qu'un enregistrement DNS, et le
  certificat HTTPS ne pourrait pas être délivré.
- Si le domaine est chez **Cloudflare**, mettre le proxy sur **DNS only**
  (nuage gris) le temps que GitHub délivre le certificat, sinon la validation échoue.

---

## 2. Côté dépôt : déclarer le domaine

Deux façons, une seule suffit.

**Option A — via l'interface GitHub (la plus simple)**
`Settings` → `Pages` → *Custom domain* → saisir `etiquettes-ecole.fr` → `Save`.
GitHub crée lui-même le fichier `CNAME` à la racine du dépôt.

**Option B — via ce dépôt**
Créer un fichier nommé `CNAME` (sans extension) à la racine, contenant
exactement une ligne :

```
etiquettes-ecole.fr
```

puis committer et pousser sur `main`.

> ⚠️ Ce fichier doit être présent **à la racine du site publié**. Le workflow
> déployant la racine du dépôt, le mettre à la racine du dépôt suffit.
> S'il disparaît, GitHub Pages perd le domaine personnalisé.

---

## 3. Attendre, puis forcer le HTTPS

- La propagation DNS prend de quelques minutes à quelques heures.
- Dans `Settings` → `Pages`, GitHub affiche la vérification du domaine puis
  « *Certificate: Active* » (délivrance Let's Encrypt automatique, gratuite).
- **Une fois le certificat actif**, cocher **`Enforce HTTPS`**.
  Cette case reste grisée tant que le certificat n'est pas délivré : c'est normal,
  il faut simplement attendre.

Résultat final :

- `https://etiquettes-ecole.fr` → le site
- `https://www.etiquettes-ecole.fr` → redirigé automatiquement vers le domaine racine
- `https://seygen.github.io/etiquettes-ecole.fr/` → redirigé vers le domaine personnalisé

---

## Vérifier que le DNS est correct

```bash
dig +short etiquettes-ecole.fr A
# doit renvoyer les quatre adresses 185.199.10x.153

dig +short www.etiquettes-ecole.fr CNAME
# doit renvoyer seygen.github.io.
```

---

## Effet sur les données déjà saisies

L'application enregistre les données dans le `localStorage` du navigateur, qui est
**cloisonné par domaine**. Une enseignante ayant déjà saisi sa classe sur
`seygen.github.io` ne la retrouvera pas automatiquement sur `etiquettes-ecole.fr`.

C'est sans gravité si le domaine est branché **avant** la diffusion du lien.
Si des personnes ont déjà utilisé l'ancienne adresse, leur indiquer d'exporter
leur liste (« Enregistrer ma liste dans un fichier ») depuis l'ancienne adresse,
puis de la recharger (« Charger ma liste ») depuis la nouvelle.
