# Jean-Claude Boniface — portfolio photographique

Site statique [Jekyll](https://jekyllrb.com/), publié par GitHub Actions sur
GitHub Pages.

- **En ligne :** https://bonantine.github.io/JCfoto/
- **Photographies :** © Jean-Claude Boniface, tous droits réservés.
- **Code :** dérivé du gabarit
  [jekyll-image-gallery-example](https://github.com/opieters/jekyll-image-gallery-example)
  d'Olivier Pieters (MIT — voir `LICENSE-CODE.md`, à conserver).

---

## Aperçu local

```sh
bundle install
bundle exec jekyll serve
```

Puis ouvrir **http://localhost:4000/JCfoto/** — le suffixe `/JCfoto/` est
nécessaire en local (`baseurl`). Redémarrer après toute modification de
`_config.yml` : ce fichier n'est pas rechargé automatiquement.

## Publier

Tout `push` sur `main` déclenche le déploiement
(`.github/workflows/pages.yml`). Aucune autre manipulation n'est nécessaire.

## Ajouter une série de photos

### 1. Générer les images web

```sh
tools/mkweb.sh ~/chemin/vers/les/originaux nom-de-la-serie "Titre affiché"
```

Le script s'occupe de tout ce qui est mécanique : il fabrique les différentes
tailles, la vignette, et écrit `_data/galleries/nom-de-la-serie.yml`.

Les **originaux pleine résolution ne sont pas copiés dans le dépôt** — seules
les versions web le sont. Le dépôt n'est donc **pas une sauvegarde** : conservez
vos originaux ailleurs.

### 2. Ajouter la carte au portfolio

Dans `_data/galleries/overview.yml`, ajouter un bloc (l'ordre du fichier =
l'ordre d'affichage, deux par ligne). Recopier le bloc `preview:` depuis le
fichier que le script vient de générer.

### 3. Créer la page

`photography/nom-de-la-serie.md` :

```markdown
---
layout: gallery
title: Titre affiché
no_menu_item: true
support: [jquery, gallery]
---
{% include gallery-layout.html gallery=site.data.galleries.nom_de_la_serie %}
```

Puis `git add`, `git commit`, `git push`.

## Modifier le texte « À propos »

`_data/translations.yml`, clé `about_text`. C'est aussi là que se trouvent les
libellés du menu. Ce fichier peut être édité directement depuis github.com : le
site se redéploie tout seul après validation.

## Changer la photo d'accueil

`_config.yml`, clés `cover_image` et `cover_alt`. L'image doit se trouver dans
`assets/photography/coverpage/`.

---

## Pièges à connaître

Quatre règles non évidentes, qui cassent le site **sans message d'erreur** :

1. **Les noms de fichiers portent les dimensions réelles.** Le gabarit relit la
   largeur dans le nom (`…-1000x668.jpg`) pour construire le `srcset`. Le nom
   doit se terminer par `-LARGEURxHAUTEUR.jpg`. `tools/mkweb.sh` le garantit —
   d'où l'intérêt de ne pas renommer à la main.

2. **`sizes:` doit rester en ordre croissant.** La visionneuse ouvre
   `sizes | last`. Un ordre décroissant afficherait la plus petite image.

3. **Aucun nom de fichier ne peut commencer par `_`.** Jekyll ignore
   silencieusement ces fichiers : le site se construit sans erreur et *toutes*
   les images sont introuvables en ligne. Les boîtiers Nikon nomment pourtant
   leurs fichiers `_DSC1234.jpg` — le script retire ce préfixe automatiquement.

4. **`.gitignore` ne liste que les tailles produites par le script.** C'est
   volontaire : un motif générique laisserait entrer les originaux pleine
   résolution et ferait gonfler l'historique du dépôt. Si vous modifiez
   `LADDER` ou `THUMB_LONG` dans `tools/mkweb.sh`, mettez `.gitignore` à jour —
   le script s'arrête avec une erreur s'il écrit un fichier exclu.

## Nom de domaine

Le site est pour l'instant sur une URL GitHub. Pour passer à un domaine propre :

1. Ajouter le domaine dans **Settings → Pages → Custom domain**. Un fichier
   `CNAME` dans le dépôt **ne suffit pas** ici, car la publication passe par un
   workflow Actions et non par une branche.
2. Configurer le DNS chez le registrar (enregistrements `A`/`AAAA` pour le
   domaine nu, ou `CNAME` vers `bonantine.github.io` pour `www`).
3. Vérifier le domaine au niveau du compte, attendre le certificat, puis activer
   **Enforce HTTPS**.
4. Enfin, remplacer `url:` dans `_config.yml` par le nouveau domaine.

`baseurl` n'a pas à changer : le workflow le recalcule à chaque build. En
revanche, oublier l'étape 4 laisse les URL canoniques pointer vers l'ancienne
adresse.
