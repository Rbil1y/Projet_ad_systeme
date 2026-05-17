## Introduction

Avec l'augmentation constante du volume de données générées dans les systèmes informatiques, la perte accidentelle de données représente l'un des risques les plus critiques pour toute infrastructure. Qu'il s'agisse d'une erreur humaine, d'une panne matérielle ou d'une corruption logicielle, l'absence d'une stratégie de sauvegarde fiable peut avoir des conséquences graves et parfois irréversibles.

Dans ce contexte, ce projet consiste à concevoir et mettre en place un système de sauvegardes automatisées locales sur une machine Ubuntu Server hébergée dans un environnement virtualisé VirtualBox. L'objectif est de créer une solution complète, fiable et autonome, capable de protéger les données de manière régulière sans intervention manuelle.

Le système repose sur trois composantes fondamentales : un script bash qui réalise la compression et l'archivage des données avec l'outil `tar`, un démon `cron` qui automatise l'exécution du script à une fréquence quotidienne, et un mécanisme de rotation qui gère automatiquement la durée de conservation des archives afin d'éviter la saturation du disque.

La solution proposée intègre également une journalisation complète des opérations, une vérification de l'intégrité des archives après leur création, ainsi qu'une procédure documentée de restauration en cas de sinistre. Ces éléments permettent d'assurer non seulement la création des sauvegardes, mais aussi leur fiabilité et leur exploitabilité réelle.

Ce projet illustre concrètement l'application des bonnes pratiques d'administration système Linux, en mettant en œuvre des outils standards du monde Unix dans un scénario réaliste de protection des données.

---