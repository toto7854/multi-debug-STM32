# FAQ & Dépannage : Débogage Multi-Cible

Ce guide de questions-réponses aborde les problèmes courants, les cas particuliers et les meilleures pratiques pour déboguer plusieurs microcontrôleurs simultanément sur un seul ordinateur.

---

## 1. Serveur GDB et Erreurs de Connexion

### Q : J'ai une erreur : "Could not start GDB server" ou "Address already in use: bind". Comment la résoudre ?
* **Cause :** Deux configurations de débogage tentent d'utiliser le même port de serveur GDB (généralement `61234` par défaut) ou le même port SWO (`61235`).
* **Solution :**
  1. Ouvrez **Debug Configurations...** pour votre deuxième/troisième projet.
  2. Accédez à l'onglet **Debugger**.
  3. Changez le **GDB Server Port** pour une valeur unique (ex. : `61236`).
  4. Changez le **SWO Port** pour une valeur unique (ex. : `61237`).
  5. Appliquez les modifications et relancez le groupe de lancement (Launch Group).

### Q : STM32CubeIDE affiche "Failed to connect to ST-LINK" ou "ST-Link USB communication error".
* **Cause A :** La sonde ST-LINK sélectionnée par son numéro de série n'est pas branchée, ou le port USB s'est mis en veille.
  * **Solution :** Cliquez sur **Scan** dans les paramètres du débogueur pour vérifier si le numéro de série est détecté. Essayez de débrancher et rebrancher la sonde.
* **Cause B :** Alimentation insuffisante. Plusieurs cartes et sondes peuvent consommer plus de courant qu'un port USB classique ne peut en fournir (généralement 500mA pour de l'USB 2.0).
  * **Solution :** Utilisez systématiquement un **Hub USB 3.0 alimenté** (avec son propre adaptateur secteur) lorsque vous déboguez plus de deux cartes. Évitez d'alimenter plusieurs cartes sur la batterie d'un ordinateur portable.

---

## 2. Bonnes Pratiques Git et Collaboration d'Équipe

### Q : Comment partager les configurations de projet dans Git sans casser le débogage pour les autres développeurs qui possèdent des sondes ST-LINK physiques différentes ?
* **Problème :** Les configurations de débogage dans Eclipse/STM32CubeIDE sont stockées dans des fichiers `.launch`. Si vous les enregistrez dans le projet et les validez sur Git, ils contiendront le numéro de série unique (S/N) de votre sonde ST-LINK. Quand un collègue récupérera le code, sa session de débogage échouera car il ne possède pas votre sonde spécifique.
* **Solutions recommandées :**
  1. **Ne pas versionner les fichiers `.launch` locaux :** Par défaut, Eclipse stocke les configurations de lancement dans le dossier local du workspace (qui n'est pas poussé sur Git). Conservez ce comportement par défaut si possible.
  2. **Configurations modèles :** Si vous souhaitez absolument partager les configurations de débogage, enregistrez-les dans le dossier du projet mais **laissez le champ ST-LINK S/N vide** dans la version validée sur Git.
     * Lors du premier lancement par un développeur, STM32CubeIDE affichera un message : *"Multiple ST-LINKs found, please select one."* (Plusieurs ST-LINK détectés, veuillez en sélectionner un).
     * Une fois qu'il aura sélectionné sa sonde locale, celle-ci sera enregistrée localement.
  3. Ajoutez les fichiers `.launch` locaux à votre fichier `.gitignore` :
     ```gitignore
     # Ignorer les configurations de débogage locales spécifiques aux développeurs
     *.launch
     ```

---

## 3. Questions Avancées

### Q : Peut-on déboguer des familles différentes de microcontrôleurs STM32 simultanément (ex. : un STM32F4 sur une carte et un STM32L4 sur une autre) ?
* **Oui, tout à fait.** Le serveur GDB de ST-LINK détecte automatiquement l'architecture du cœur cible (Cortex-M0+, M4, M7, etc.) et utilise les registres correspondants. Vous devez simplement veiller à ce que chaque configuration de débogage pointe vers le bon binaire et le bon matériel cible.

### Q : Est-ce que le fait d'arrêter ou de redémarrer une session de débogage affecte les autres ?
* **Non.** Chaque session s'exécute dans sa propre instance de serveur GDB sur un port réseau distinct. Vous pouvez arrêter, compiler, flasher et redémarrer le `Projet_IHM` sans interrompre le `Projet_Principal`.
* > [!TIP]
  > Cette indépendance est très utile pour tester la **robustesse** du système : vous pouvez suspendre un microcontrôleur (simulant un crash ou une perte de communication) et observer comment la machine d'état de l'autre carte gère cette perte.

### Q : Mon débogueur est lent, ou les Live Expressions mettent du temps à s'actualiser.
* **Cause :** Surcharge du serveur GDB due à la surveillance d'un trop grand nombre de variables ou de signaux simultanés.
* **Solutions :**
  1. **Augmenter l'intervalle de rafraîchissement :** Faites un clic droit dans l'onglet **Live Expressions**, sélectionnez *Refresh Interval...*, et augmentez la valeur par défaut de `200ms` à `500ms` ou `1000ms`.
  2. **Limiter les expressions :** Supprimez de la vue Live Expressions les variables que vous n'avez plus besoin de surveiller en direct.
  3. **Désactiver le traçage SWO :** Si vous n'utilisez pas activement la sortie SWO (Serial Wire Output) pour rediriger des `printf`, désactivez-la dans l'onglet **Debugger** de votre configuration de débogage pour économiser de la bande passante.

### Q : Comment fonctionne la réinitialisation (Reset) ? Si je clique sur le bouton "Restart", est-ce que cela réinitialise toutes les cartes ?
* **Non.** Le clic sur le bouton de redémarrage (icône flèche jaune play/pause) ou de réinitialisation (Reset) dans le panneau de contrôle ne réinitialise que le microcontrôleur associé au *thread actuellement sélectionné* dans la vue **Debug**.
* Pour réinitialiser toutes les cartes, sélectionnez chaque thread un par un dans la vue Debug et cliquez sur le bouton Reset correspondante.
