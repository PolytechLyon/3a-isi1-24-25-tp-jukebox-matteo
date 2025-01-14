# Compte Rendu TP Jukebox

## Binôme
- BONTEMPS Flavien
- CHAUCHET Mattéo

## Choix de conception et de réalisation
#### Structure globale:
  Le projet a été conçu en utilisant Vue 3 avec le mode script setup, permettant une gestion claire et centralisée de l'état de l'application. Les principales fonctionnalités incluent :
  - Ajout de pistes par URL ou par chargement de fichier.
  - Lecture, pause et navigation entre les pistes.
  - Gestion des modes de lecture (répétition de la piste, de la playlist ou sans répétition).
  - Validation des URLs pour éviter l'ajout de fichiers non pris en charge.
  - Sauvegarde et restauration de la playlist via localStorage.

#### Composants clés:
  ##### Gestion des pistes:
  - Une fonction addTrack permet l'ajout de pistes en validant leur origine (URL ou fichier).
  - Les pistes invalides sont marquées visuellement et non jouables.
  ##### Lecture audio:
  - Une instance de Audio est utilisée pour gérer la lecture et les événements audio.
  - Les événements timeupdate et ended sont gérés pour synchroniser l'interface et la logique.
  ##### Contrôles de l'utilisateur:
  - L'interface inclut des boutons de lecture/pause, des barres de progression interactives et des options de mode de lecture.
  - Les éléments invalides sont signalés par un style distinct (texte barré en rouge).
  ##### Persistance des données:
  - La playlist est sauvegardée dans le localStorage pour être restaurée au prochain chargement.

## Difficultés rencontrées (optionnel)
  #### 1. Validation des URLs
Problème : Certaines URLs valides étaient marquées comme invalides en raison de restrictions du navigateur ou de la source.
Solution : Améliorer la gestion des erreurs en utilisant également les événements onloadedmetadata pour une validation supplémentaire.
  #### 2. La création de la barre de lecture sans utilisé
Problème : 
Solution : 

## Extensions réalisées (optionnel)
