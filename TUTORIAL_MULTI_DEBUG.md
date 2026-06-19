# Tutoriel Étape par Étape : Débogage Multi-Cible dans STM32CubeIDE

Ce guide vous accompagne dans la configuration requise pour déboguer plusieurs microcontrôleurs simultanément à l'aide de plusieurs sondes ST-LINK/V2 connectées à un seul PC.

---

## Prérequis
* **STM32CubeIDE** (v1.10.0 ou ultérieure recommandée).
* **STM32CubeProgrammer** installé (intégré à STM32CubeIDE, mais la version autonome est utile).
* Plusieurs **sondes ST-LINK/V2** (ou débogueurs ST-LINK intégrés sur les cartes Nucleo/Discovery).
* Plusieurs cartes cibles (par exemple, vos cartes IHM, Principale et de Puissance).
* **Câbles Micro-USB / Mini-USB** pour chaque sonde.

---

## Étape 1 : Configuration Physique & Étiquetage des Sondes

Pour éviter toute confusion lors du débogage, vous devez identifier physiquement quelle sonde ST-LINK est connectée à quelle carte microcontrôleur.

1. Connectez chaque sonde ST-LINK à sa carte cible correspondante.
2. Étiquetez chaque sonde ST-LINK physique avec une étiqueteuse ou du ruban adhésif (ex. : `"Sonde A - IHM"`, `"Sonde B - Principale"`).
3. Connectez toutes les sondes à votre PC. 
   > [!TIP]
   > Si vous manquez de ports USB, utilisez un **Hub USB 3.0 alimenté** (avec alimentation externe). Les hubs non alimentés peuvent ne pas fournir suffisamment de courant pour alimenter plusieurs sondes ST-LINK et microcontrôleurs cibles simultanément.

---

## Étape 2 : Récupérer les Numéros de Série des Sondes

Chaque sonde ST-LINK possède un numéro de série matériel unique (S/N). Nous avons besoin de cet identifiant pour indiquer à STM32CubeIDE quel serveur GDB contrôle quelle sonde physique.

### Méthode A : Via l'interface graphique de STM32CubeProgrammer (Le plus simple)
1. Ouvrez **STM32CubeProgrammer**.
2. Dans le panneau de droite, réglez le type de connexion sur **ST-LINK**.
3. Localisez le champ déroulant **Serial number**.
4. Cliquez sur le bouton de rafraîchissement/scan (flèches circulaires).
5. Toutes les sondes ST-LINK connectées apparaîtront dans la liste déroulante.
6. Copiez les numéros de série et associez-les à vos étiquettes (ex. : `066EFF535051717867152438` correspond à la **Sonde A - IHM**).

### Méthode B : Via la ligne de commande STM32CubeProgrammer (CLI)
Ouvrez une invite de commande (ou PowerShell) et accédez à votre répertoire d'installation de STM32CubeProgrammer, puis exécutez :
```powershell
# Lister toutes les sondes ST-LINK connectées
& "C:\ST\STM32CubeProgrammer\bin\STM32_Programmer_CLI.exe" --list
```
Cela affichera la liste de toutes les sondes détectées avec leur index et leur **Numéro de Série (Serial Number)**.

---

## Étape 3 : Configurer les Configurations de Débogage dans STM32CubeIDE

Vous devez configurer une configuration de débogage (Debug Configuration) distincte pour chaque projet. Voici comment les paramétrer pour éviter tout conflit.

Supposons que nous ayons deux projets : **Projet_IHM** et **Projet_Principal**.

### Configuration pour le Projet 1 (Projet_IHM)
1. Dans STM32CubeIDE, faites un clic droit sur **Projet_IHM** -> **Debug As** -> **Debug Configurations...**
2. Sélectionnez **STM32 Cortex-M C/C++ Application** et cliquez sur l'icône **New Launch Configuration** (Nouvelle configuration de lancement).
3. Nommez-la : `Projet_IHM_Debug`.
4. Allez dans l'onglet **Debugger** :
   * **Debug Probe :** Sélectionnez `ST-LINK (ST-LINK GDB Server)`.
   * Sous **Interface :** Sélectionnez votre interface (généralement `SWD`).
   * Sous **Connection Settings** (Paramètres de connexion) :
     * Cochez **Shared ST-LINK**.
     * Cherchez **ST-LINK S/N** et cliquez sur le bouton **Scan**.
     * Sélectionnez le numéro de série qui correspond à la **Sonde A - IHM** dans la liste déroulante.
     * **GDB Server Port :** Laissez par défaut à `61234`.
     * **SWO Port :** Laissez par défaut à `61235`.
5. Cliquez sur **Apply** (Appliquer).

---

### Configuration pour le Projet 2 (Projet_Principal)
1. Dans la fenêtre Debug Configurations, sélectionnez à nouveau **STM32 Cortex-M C/C++ Application** -> Créez une autre configuration de lancement.
2. Nommez-la : `Projet_Principal_Debug`.
3. Allez dans l'onglet **Debugger** :
   * **Debug Probe :** Sélectionnez `ST-LINK (ST-LINK GDB Server)`.
   * Sous **Interface :** Sélectionnez `SWD`.
   * Sous **Connection Settings** :
     * Cochez **Shared ST-LINK**.
     * Cliquez sur **Scan** à côté de **ST-LINK S/N**.
     * Sélectionnez le numéro de série correspondant à la **Sonde B - Principale**.
     * > [!IMPORTANT]
       * **GDB Server Port :** Modifiez cette valeur par **`61236`** (Incrémentée pour éviter tout conflit).
       * **SWO Port :** Modifiez cette valeur par **`61237`** (Incrémentée pour éviter tout conflit).
4. Cliquez sur **Apply**.

---

### Guide d'Allocation des Ports pour Multi-Cibles

Si vous avez plus de deux cartes, continuez à incrémenter les ports GDB et SWO :

| Nom de la Cible | Sélection ST-LINK S/N | Port GDB | Port SWO |
| :--- | :--- | :--- | :--- |
| Cible 1 (IHM) | `Numéro de série Sonde A` | **61234** | **61235** |
| Cible 2 (Principale) | `Numéro de série Sonde B` | **61236** | **61237** |
| Cible 3 (Puissance) | `Numéro de série Sonde C` | **61238** | **61239** |

---

## Étape 4 : Groupe de Lancement (Débogage Multi-Cartes en un Clic)

Au lieu de lancer manuellement chaque configuration de débogage une par une, vous pouvez les regrouper pour qu'elles démarrent automatiquement dans l'ordre.

1. Dans STM32CubeIDE, allez dans **Debug Configurations...**
2. Double-cliquez sur **Launch Group** (en bas de la liste de gauche) pour créer un nouveau groupe.
3. Nommez-le : `System_Multi_Debug_Group`.
4. Cliquez sur le bouton **Add...** (Ajouter) à droite :
   * Sélectionnez `Projet_IHM_Debug`.
   * **Action :** Sélectionnez `Launch` (Lancer).
   * **Post-launch action :** Sélectionnez `Wait for console output` ou `Delay` (ex. : `2` secondes) pour laisser le serveur GDB démarrer avant de lancer le suivant.
5. Cliquez à nouveau sur le bouton **Add...** :
   * Sélectionnez `Projet_Principal_Debug`.
   * **Action :** Sélectionnez `Launch`.
6. Cliquez sur **Apply** puis sur **Debug**.

---

## Étape 5 : Gérer la Session de Débogage Multi-Cible

Une fois démarré, STM32CubeIDE passera en perspective de débogage (**Debug Perspective**). Voici comment s'y retrouver :

### 1. La Vue Debug (Pile d'Appels / Call Stack)
Ce panneau liste toutes les sessions actives. Vous pouvez les développer pour voir la pile d'appels de chaque microcontrôleur :
* `Projet_IHM_Debug [STM32 Cortex-M C/C++ Application]` -> Thread #1 (Running/Suspended)
* `Projet_Principal_Debug [STM32 Cortex-M C/C++ Application]` -> Thread #1 (Running/Suspended)

*Sélectionnez* un thread dans cette vue pour mettre à jour la fenêtre de l'éditeur de code, la vue **Variables** et la vue **Live Expressions** afin qu'elles affichent les données du microcontrôleur sélectionné.

### 2. Basculer entre les Consoles
Sous l'onglet **Console**, vous verrez la sortie de tous les serveurs GDB. Utilisez l'icône de liste déroulante **Display Selected Console** (un écran bleu) pour basculer entre les journaux du débogueur IHM et ceux du débogueur Principal.

### 3. Contrôler l'Exécution
Vous pouvez contrôler les microcontrôleurs indépendamment ou ensemble :
* **Suspendre/Reprendre une cible individuelle :** Cliquez d'abord sur la cible dans la *Vue Debug*, puis cliquez sur **Resume** (F8) ou **Suspend**.
* **Points d'arrêt globaux :** Si vous placez un point d'arrêt dans `ihm.cpp`, il ne se déclenchera que lorsque la carte IHM atteindra cette ligne. Un point d'arrêt dans `principal.cpp` ne se déclenchera que lorsque la carte Principale l'atteindra.

---

*Passez au **[Guide de FAQ & Dépannage](FAQ_TROUBLESHOOTING.md)** pour gérer les problèmes fréquents.*
