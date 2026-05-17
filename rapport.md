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