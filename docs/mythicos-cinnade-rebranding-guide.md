# Guide de fork : passer de Cinnamon/Linux Mint à Cinnade/MythicOS

Ce document donne une feuille de route concrète pour rebrander et forker Cinnamon de manière propre, afin que l'identité visible soit **Cinnade** (environnement de bureau) et **MythicOS** (distribution/OS) plutôt que Cinnamon/Linux Mint.

## 1) Définir l'identité cible (avant de coder)

Décide des noms qui seront utilisés partout :

- Nom du shell desktop : `Cinnade`
- Nom du projet upstream dans le code (si conservé) : `cinnamon` (interne transitoire)
- Nom distribution : `MythicOS`
- Session X11 : `cinnade`
- Session Wayland : `cinnade-wayland`
- Schéma d'identifiants (DBus, desktop, gsettings) :
  - Préfixe recommandé : `org.mythicos.cinnade`

> Conseil : fais un tableau de correspondance "ancien -> nouveau" et garde une couche de compatibilité pendant 1-2 versions.

## 2) Rebranding visible utilisateur (priorité haute)

### 2.1 Sessions et fichiers desktop

À renommer/dupliquer :

- `cinnamon.session` / `cinnamon-wayland.session` -> `cinnade.session` / `cinnade-wayland.session`
- Entrées `.desktop` de session affichées dans le greeter (GDM/SDDM/LightDM)

Objectif : l'utilisateur ne voit plus "Cinnamon" dans l'écran de connexion.

### 2.2 Branding UI

Remplacer les chaînes visibles :

- "Cinnamon" -> "Cinnade"
- "Linux Mint" -> "MythicOS"

Zones à vérifier :

- boîtes "À propos"
- noms de panneau/menu
- notifications système
- libellés d'outils (`cinnamon-settings`, `cinnamon-menu-editor`, etc.)
- métadonnées des applets/desklets/extensions

### 2.3 Icônes, logos, assets

- Créer un set d'icônes `cinnade*` (symbolic + color)
- Remplacer badges/logo Cinnamon dans `files/usr/share/icons/...`
- Remplacer assets Mint éventuels (wallpapers, marqueurs, etc.)

## 3) Rebranding technique (moyenne/haute complexité)

## 3.1 Identifiants applicatifs

Migrer progressivement :

- `org.cinnamon.*` -> `org.mythicos.cinnade.*`
- noms de binaire (en gardant des wrappers de compatibilité) :
  - `cinnamon` -> `cinnade`
  - `cinnamon-settings` -> `cinnade-settings`

Important : évite une rupture brutale. Garde des symlinks/wrappers :

- anciens binaires -> nouveaux binaires
- anciens DBus names (si possible) redirigés

## 3.2 Schémas GSettings

- Dupliquer les schémas `org.cinnamon.*` vers `org.mythicos.cinnade.*`
- Prévoir migration automatique des clés utilisateur au premier login
- Conserver lecture fallback des anciennes clés pendant transition

## 3.3 D-Bus et services

- Renommer services `.service` et interfaces si nécessaire
- Maintenir alias de compatibilité pour outils/plugins existants

## 3.4 Chemins et répertoires installés

Remplacer au fil de l'eau :

- `/usr/share/cinnamon` -> `/usr/share/cinnade` (avec symlink)
- noms de paquets, scripts, utilitaires

## 4) Packaging distribution MythicOS

Dans les paquets (Debian/RPM/PKGBUILD selon ton socle) :

- Renommer métapaquets : `cinnamon` -> `cinnade`
- `Provides/Replaces/Conflicts` pour transition propre
- Adapter postinst/prerm pour migration de config
- Vérifier dépendances de tous les composants satellites

Exemple de stratégie paquets :

1. version N : paquets `cinnade-*` + compat `cinnamon-*`
2. version N+1 : compat maintenue mais dépréciée
3. version N+2 : retrait progressif des alias legacy

## 5) Internationalisation (i18n)

Après rebranding des chaînes :

- Mettre à jour catalogues `po/`
- Régénérer `.pot`
- Marquer chaînes renommées pour traducteurs

## 6) Compatibilité écosystème Cinnamon

Si tu veux garder applets/extensions existantes :

- Maintenir API JavaScript stable (`imports.ui`, `imports.misc`)
- Conserver répertoire des extensions existantes ou offrir un chemin de migration
- Documenter clairement ce qui casse

## 7) Sécurité et maintenance du fork

- Conserver un remote upstream (Cinnamon)
- Faire un merge/rebase régulier des correctifs sécurité
- Isoler les commits de branding des commits fonctionnels
- Automatiser les vérifications de chaîne interdite : `Linux Mint`, `Cinnamon`

## 8) Checklist opérationnelle pour ton dépôt

1. Ajouter un script de scan branding (CI) :
   - échec CI si "Linux Mint" ou "Cinnamon" trouvé dans chaînes UI non autorisées
2. Renommer sessions
3. Renommer exécutables principaux + wrappers
4. Migrer schémas gsettings + script migration utilisateur
5. Rebranding icônes/desktop files
6. Vérifier login, panel, menu, settings, extensions
7. Publier notes de migration

## 9) Commandes utiles pour démarrer dans ce repo

Audit rapide des occurrences :

```bash
rg -n "Linux Mint|Cinnamon|org\.cinnamon|cinnamon-" js files data debian
```

Lister les fichiers de session/desktop à traiter :

```bash
rg --files | rg "session|cinnamon.*desktop|org\.cinnamon|cinnamon-"
```

## 10) Stratégie recommandée (pragmatique)

- **Phase 1 (branding visuel)** : chaînes UI + sessions + logos
- **Phase 2 (identifiants techniques)** : DBus/GSettings/binaires avec compat
- **Phase 3 (nettoyage)** : retrait progressif des alias legacy

Cette approche te permet de livrer vite une identité MythicOS/Cinnade sans casser l'écosystème dès la première version.
