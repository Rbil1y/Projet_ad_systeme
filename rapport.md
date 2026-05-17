## Introduction

Avec l'augmentation constante du volume de données générées dans les systèmes informatiques, la perte accidentelle de données représente l'un des risques les plus critiques pour toute infrastructure. Qu'il s'agisse d'une erreur humaine, d'une panne matérielle ou d'une corruption logicielle, l'absence d'une stratégie de sauvegarde fiable peut avoir des conséquences graves et parfois irréversibles.

Dans ce contexte, ce projet consiste à concevoir et mettre en place un système de sauvegardes automatisées locales sur une machine Ubuntu Server hébergée dans un environnement virtualisé VirtualBox. L'objectif est de créer une solution complète, fiable et autonome, capable de protéger les données de manière régulière sans intervention manuelle.

Le système repose sur trois composantes fondamentales : un script bash qui réalise la compression et l'archivage des données avec l'outil `tar`, un démon `cron` qui automatise l'exécution du script à une fréquence quotidienne, et un mécanisme de rotation qui gère automatiquement la durée de conservation des archives afin d'éviter la saturation du disque.

La solution proposée intègre également une journalisation complète des opérations, une vérification de l'intégrité des archives après leur création, ainsi qu'une procédure documentée de restauration en cas de sinistre. Ces éléments permettent d'assurer non seulement la création des sauvegardes, mais aussi leur fiabilité et leur exploitabilité réelle.

Ce projet illustre concrètement l'application des bonnes pratiques d'administration système Linux, en mettant en œuvre des outils standards du monde Unix dans un scénario réaliste de protection des données.

---
## Objectifs du projet

### Objectif général

L'objectif principal de ce projet est de concevoir et déployer, sur un serveur Ubuntu 22.04 LTS virtualisé sous VirtualBox, un système de sauvegardes automatisées locales capable d'archiver des données critiques, de planifier leur exécution de manière autonome, d'assurer leur rotation et de permettre leur restauration complète en cas de perte.

### Objectifs spécifiques

**Préparer l'environnement de travail.** Créer une arborescence de répertoires structurée comprenant un répertoire source `/data`, un répertoire de destination `/backups` et un répertoire de journaux `/var/log/backups`. Générer des fichiers de test représentatifs simulant des données réelles à protéger.

**Développer un script de sauvegarde robuste.** Écrire un script bash complet qui compresse le contenu de `/data` avec `tar` et `gzip`, nomme les archives avec un horodatage précis, vérifie l'intégrité de l'archive après sa création et consigne toutes les opérations dans un fichier de log avec niveaux de sévérité.

**Gérer les erreurs et anticiper les problèmes.** Intégrer dans le script des vérifications préliminaires portant sur l'existence du répertoire source, l'espace disque disponible et la définition explicite du `PATH` pour assurer la compatibilité avec l'environnement cron.

**Automatiser l'exécution avec cron.** Configurer une tâche planifiée dans le crontab de root pour exécuter le script automatiquement chaque jour à 2h00 du matin, et vérifier son bon fonctionnement par un test à court terme.

**Mettre en place la rotation des sauvegardes.** Implémenter un mécanisme de suppression automatique des archives de plus de 7 jours, afin de conserver un historique raisonnable tout en maîtrisant l'espace disque consommé.

**Valider par un test de restauration.** Simuler un scénario de perte de données par suppression de `/data`, puis restaurer l'intégralité des fichiers depuis la sauvegarde la plus récente, et vérifier que les données récupérées sont intactes.

---

## Problématique

Dans un environnement informatique, la continuité des données est une priorité absolue. Pourtant, les opérations de sauvegarde sont souvent négligées, effectuées de manière irrégulière ou confiées entièrement à l'action humaine, ce qui les rend peu fiables sur le long terme.

Sur un serveur Linux administré en ligne de commande, la mise en place d'un système de sauvegarde efficace soulève plusieurs défis pratiques. D'un côté, le script doit être suffisamment robuste pour gérer les erreurs, vérifier l'intégrité des archives et journaliser chaque opération. De l'autre, l'environnement d'exécution automatique via cron est plus restrictif que la session interactive d'un utilisateur : le `PATH` n'y est pas le même, les variables d'environnement sont limitées, et une erreur silencieuse peut passer inaperçue pendant des jours.

S'ajoutent à cela des problèmes de gestion des permissions : un script exécuté par root doit manipuler des fichiers appartenant à l'utilisateur `jose`, et les archives restaurées peuvent changer de propriétaire si les droits ne sont pas réajustés après la restauration. La rotation des sauvegardes doit également être soigneusement calibrée pour ne supprimer que les archives réellement trop anciennes, sans risque de supprimer une archive encore récente par une mauvaise interprétation des dates de modification.

Ainsi, la problématique centrale de ce projet peut être formulée comme suit : comment concevoir et déployer, sur un serveur Ubuntu 22.04 LTS, un système de sauvegardes automatisées locales qui soit fiable, autonome, résistant aux erreurs courantes et capable de restaurer les données dans leur état d'origine en cas de sinistre ?

---

## Méthodologie

**Analyse des besoins.** Identifier les données à protéger et définir les exigences du système : fréquence de sauvegarde (quotidienne), durée de rétention (7 jours), format d'archivage (tar + gzip), niveau de journalisation souhaité et procédure de restauration à documenter.

**Conception de l'architecture de stockage.** Définir l'organisation des répertoires : `/data` pour les données sources, `/backups` pour les archives compressées, `/var/log/backups` pour les journaux d'exécution. Fixer les conventions de nommage des archives avec horodatage au format `backup_YYYY-MM-DD_HH-MM-SS.tar.gz`.

**Développement du script bash.** Écrire le script `/usr/local/bin/backup.sh` de manière itérative : d'abord les variables de configuration, ensuite la fonction de journalisation, puis les vérifications préliminaires, la création de l'archive, la vérification d'intégrité et enfin la rotation. Ajouter en tête du script un `PATH` explicite pour garantir la compatibilité avec cron.

**Tests manuels et débogage.** Exécuter le script manuellement avec `sudo /usr/local/bin/backup.sh` et vérifier chaque étape : existence de l'archive créée, contenu du log, messages d'erreur éventuels. Corriger les problèmes de permissions en utilisant le nom d'utilisateur explicite `jose` plutôt que la variable `$USER` avec sudo.

**Planification via cron.** Ajouter la tâche dans le crontab de root avec `sudo crontab -e`. Effectuer un test à court terme en configurant une exécution à la minute suivante, puis vérifier la création d'une nouvelle archive avant de rétablir la planification quotidienne à 2h00.

**Simulation de la rotation.** Créer des archives fictives datées artificiellement dans le passé avec `touch -d` pour déclencher le mécanisme de suppression et vérifier qu'il opère correctement sur les archives de plus de 7 jours.

**Test de restauration.** Supprimer intentionnellement le contenu de `/data` pour simuler un sinistre, identifier la sauvegarde la plus récente, extraire l'archive dans un répertoire temporaire, copier les fichiers vers `/data` et vérifier le nombre et le contenu des fichiers récupérés.

---

## Outils et technologies utilisés

Dans le cadre de la réalisation de ce projet, plusieurs outils et technologies ont été mis en œuvre pour concevoir, développer et tester le système de sauvegardes automatisées.

**Environnement de virtualisation.** L'ensemble du projet est réalisé sur une machine virtuelle Ubuntu Server 22.04 LTS hébergée dans Oracle VirtualBox. L'adresse IP de la machine est `192.168.1.212` et le compte utilisateur principal est `jose`. VirtualBox permet d'isoler l'environnement de test sans affecter le système hôte.

**Système d'exploitation.** Ubuntu Server 22.04 LTS est utilisé sans interface graphique. Toutes les opérations sont effectuées en CLI, ce qui correspond aux conditions réelles d'administration d'un serveur Linux en production.

**bash (Bourne Again SHell).** Langage de script utilisé pour développer le script principal `backup.sh`. Il permet de combiner des commandes système, des structures de contrôle, des fonctions et la gestion des codes de retour pour construire un programme robuste.

**tar (Tape ARchive).** Outil fondamental sous Linux pour créer des archives de fichiers. Utilisé avec l'option `-czf` pour compresser en gzip et créer des archives `.tar.gz`, et avec `-tzf` pour vérifier leur contenu sans les extraire.

**gzip.** Algorithme de compression intégré à tar via l'option `-z`. Réduit significativement la taille des archives, ce qui optimise l'espace disque utilisé par les sauvegardes.

**cron.** Démon système Unix qui exécute des commandes planifiées selon une syntaxe à cinq champs (minute, heure, jour du mois, mois, jour de la semaine). Il constitue le moteur d'automatisation du projet, permettant l'exécution quotidienne du script sans intervention humaine.

**find.** Commande Linux utilisée dans le script pour identifier les archives dont la date de modification dépasse 7 jours (`-mtime +7`), dans le cadre du mécanisme de rotation.

**nano.** Éditeur de texte en ligne de commande utilisé pour créer et éditer le script bash ainsi que le crontab. Accessible et adapté aux débutants, il est invoqué systématiquement avec `sudo` pour les fichiers système.

**systemd / journalctl.** Le service cron est géré par systemd sur Ubuntu 22.04. La commande `systemctl status cron` permet de vérifier son état, et `journalctl` peut être utilisé pour consulter les journaux système liés à son exécution.

---

## Description du projet

### Projet 2 — Sauvegardes Automatisées Locales sur Ubuntu Server

Ce projet consiste à concevoir et déployer un système complet de sauvegardes automatisées sur un serveur Ubuntu 22.04 LTS virtualisé. L'objectif est de protéger les données contenues dans le répertoire `/data` en produisant des archives compressées, stockées localement, renouvelées quotidiennement et conservées pendant 7 jours.

**Concept du projet.** Le système repose sur un script bash central qui orchestre l'ensemble du processus : vérification de l'environnement, création de l'archive, contrôle d'intégrité, journalisation et rotation. Ce script est invoqué automatiquement par le démon cron, ce qui garantit son exécution sans dépendre d'une action humaine.

**Organisation du système.** Le projet est structuré autour de trois répertoires principaux.

Le répertoire `/data` constitue la source de données à protéger. Il contient une arborescence de fichiers répartis en quatre sous-répertoires : `documents/` pour les fichiers texte et rapports, `configs/` pour les fichiers de configuration applicatifs, `images/` pour les fichiers binaires simulant des données multimédia, et `logs_app/` pour les journaux d'application. Au total, 9 fichiers de test y sont générés pour représenter un ensemble de données réaliste.

Le répertoire `/backups` est la destination des archives créées par le script. Chaque sauvegarde y est stockée sous la forme d'un fichier `.tar.gz` dont le nom intègre un horodatage précis au format `backup_YYYY-MM-DD_HH-MM-SS.tar.gz`, permettant d'identifier immédiatement la date et l'heure de sa création. La rotation automatique supprime les archives dont la date de modification dépasse 7 jours.

Le répertoire `/var/log/backups` centralise les journaux d'exécution. Le fichier `backup.log` enregistre chaque opération avec son horodatage et son niveau de sévérité (`[INFO]`, `[OK]`, `[ERREUR]`), ce qui permet de diagnostiquer rapidement tout incident et de prouver le bon fonctionnement du système.

**Fonctionnement du script.** À chaque exécution, `backup.sh` suit un enchaînement précis. Il commence par définir un `PATH` explicite pour garantir la disponibilité des commandes en environnement cron. Il vérifie ensuite l'existence de `/data` et la disponibilité d'au moins 100 Mo d'espace disque libre. Il crée l'archive compressée avec `tar -czf`, contrôle immédiatement son intégrité avec `tar -tzf`, journalise le résultat, puis déclenche la rotation en supprimant les archives trop anciennes avec `find -mtime +7`. Un résumé final indique le nombre total d'archives conservées et l'espace occupé.

**Planification et automatisation.** La tâche cron est enregistrée dans le crontab de root avec la syntaxe `0 2 * * * /usr/local/bin/backup.sh >> /var/log/backups/cron.log 2>&1`, assurant une exécution quotidienne à 2h00 du matin avec redirection complète des sorties vers un fichier de log dédié.

**Procédure de restauration.** En cas de perte de données, la restauration s'effectue en identifiant d'abord l'archive la plus récente avec `ls -t /backups/backup_*.tar.gz | head -1`, en vérifiant son contenu avec `tar -tzf`, puis en l'extrayant dans un répertoire temporaire avant de copier les fichiers vers `/data`. Les permissions sont ensuite réajustées avec `sudo chown -R jose:jose /data` pour garantir l'accès à l'utilisateur.

---
## Résultats et analyse

Après la mise en place complète du système sur la machine virtuelle (IP `192.168.1.212`, utilisateur `jose`), le projet a produit les résultats suivants.

L'environnement de travail a été correctement initialisé : les répertoires `/data`, `/backups` et `/var/log/backups` ont été créés avec les permissions adaptées à l'utilisateur `jose`. Les 9 fichiers de test représentant environ 440 Ko de données ont été générés avec succès dans les quatre sous-répertoires de `/data`.

Le script `backup.sh` a été développé, déposé dans `/usr/local/bin/` et rendu exécutable. Son premier lancement manuel a confirmé le bon enchaînement de toutes les étapes : vérifications préliminaires validées, archive créée et nommée correctement avec horodatage, intégrité vérifiée par `tar -tzf`, log produit avec les niveaux `[INFO]` et `[OK]`, et rotation exécutée sans erreur.

La tâche cron a été configurée dans le crontab de root et son fonctionnement a été validé par un test à court terme. Une deuxième archive est apparue dans `/backups/` après l'exécution automatique, confirmant que le planificateur opère correctement.

La simulation de rotation a été réalisée en créant 10 archives fictives datées de 1 à 10 jours en arrière. Après exécution du script, seules les archives de moins de 7 jours ont été conservées, les trois plus anciennes ayant été supprimées automatiquement. Cette suppression a été tracée dans le fichier `backup.log`.

Le test de restauration a été concluant : après suppression intentionnelle du contenu de `/data`, les 9 fichiers ont été intégralement récupérés depuis la sauvegarde la plus récente. La vérification du contenu du fichier `rapport.txt` a confirmé que les données étaient intactes.

**Analyse du fonctionnement.**

Points positifs : le script est modulaire et facilement configurable grâce à ses variables centralisées. La journalisation horodatée permet un suivi précis des opérations. L'intégration du `PATH` explicite résout de manière pérenne le problème le plus fréquent des scripts lancés par cron. La rotation par `find -mtime` est fiable et ne nécessite aucune maintenance manuelle.

Limites observées : le système de sauvegarde est local, ce qui signifie qu'une panne du disque dur hébergeant `/backups` entraînerait une perte simultanée des données sources et des sauvegardes. Par ailleurs, le script ne gère pas les sauvegardes incrémentielles, ce qui implique la copie complète de `/data` à chaque exécution, même si les données n'ont pas changé. Enfin, l'absence d'alerte par courriel ou par notification en cas d'échec rend le système silencieux lors d'un incident.

Améliorations possibles : ajouter un mécanisme d'envoi d'alerte par courriel en cas d'erreur (`mail` ou `sendmail`). Implémenter des sauvegardes incrémentielles avec `rsync` pour optimiser le temps d'exécution et l'espace utilisé. Envisager une copie des archives vers un support externe ou un serveur distant pour assurer la redondance. Améliorer la sécurité des archives en appliquant un chiffrement avec `gpg` avant le stockage.

---
## Perspectives d'évolution du projet

Dans une perspective d'amélioration, ce projet de sauvegardes automatisées peut évoluer vers une solution encore plus robuste et adaptée aux exigences d'un environnement de production réel.

**Mise en place de sauvegardes distantes.** La copie des archives sur un serveur distant via `rsync` ou `scp` permettrait d'assurer la redondance géographique des données et d'éliminer le risque de perte totale en cas de défaillance du disque local. Une connexion SSH sans mot de passe avec clé publique rendrait cette opération automatisable sans intervention humaine.

**Sauvegardes incrémentielles avec rsync.** Plutôt que de copier l'intégralité de `/data` à chaque exécution, l'utilisation de `rsync --link-dest` permettrait de ne transférer que les fichiers modifiés depuis la dernière sauvegarde. Cette approche réduit significativement le temps d'exécution et l'espace disque consommé, tout en conservant un historique complet grâce aux liens matériels.

**Alertes et notifications automatiques.** Intégrer un mécanisme d'alerte par courriel (`mailutils`, `sendmail`) pour notifier l'administrateur en cas d'échec de sauvegarde. Cela permettrait de détecter immédiatement un problème sans avoir à consulter manuellement les logs, ce qui est essentiel dans un contexte de production.

**Chiffrement des archives.** Appliquer un chiffrement symétrique ou asymétrique des archives avec `gpg` avant leur stockage ou leur transfert. Cela garantirait la confidentialité des données sauvegardées, notamment en cas de vol ou d'accès non autorisé au serveur de stockage.

**Interface de supervision.** Développer un tableau de bord simple accessible via un navigateur ou un script de rapport automatique permettant de visualiser l'état des sauvegardes, l'historique des exécutions et les éventuelles erreurs détectées dans les logs.

---

## Conclusion

Ce projet de sauvegardes automatisées locales, réalisé sur un serveur Ubuntu Server 22.04 LTS virtualisé sous VirtualBox, a permis de mettre en pratique les concepts fondamentaux de l'administration système Linux et de la protection des données.

À travers la conception et le déploiement d'un système complet articulé autour d'un script bash, d'un planificateur cron et d'un mécanisme de rotation, nous avons démontré comment automatiser de manière fiable et autonome la protection d'un ensemble de données critiques. Chaque composante du système remplit un rôle précis : le script orchestre les opérations et gère les erreurs, cron garantit l'exécution régulière sans intervention humaine, et la rotation maintient un historique raisonnable sans saturer le disque.

Le projet a également mis en lumière plusieurs problèmes courants liés à l'administration Linux en ligne de commande, notamment la gestion des permissions avec sudo, le comportement de la variable `$USER` dans un contexte d'élévation de privilèges, et l'importance du `PATH` explicite dans les scripts exécutés par cron. Ces difficultés, anticipées et documentées tout au long du projet, font partie intégrante des compétences d'un administrateur système.

Le test de restauration, étape souvent négligée dans les stratégies de sauvegarde, a confirmé que le système ne se contente pas de créer des archives mais permet effectivement de récupérer les données dans leur état d'origine. C'est cette capacité de restauration qui donne tout son sens à la sauvegarde.

En conclusion, ce projet démontre que des outils standards et accessibles — bash, tar, cron — suffisent pour construire une solution de sauvegarde solide et professionnelle, à condition d'être combinés avec méthode, rigueur et une bonne gestion des cas d'erreur.
