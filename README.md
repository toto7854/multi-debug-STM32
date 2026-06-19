# Débogage Multi-Cible avec Plusieurs Sondes ST-LINK/V2 dans STM32CubeIDE

Ce dépôt contient un guide pour configurer et exécuter des sessions de débogage simultanées pour plusieurs microcontrôleurs STM32 sur un seul PC.

## À quoi ça sert ?
Lorsque l'on travaille sur des systèmes multi-cartes (par exemple, des cartes IHM, Principale et de Puissance qui interagissent), le débogage de la communication inter-cartes (SPI, I2C, CAN, UART) est complexe. Cette configuration vous permet de :
- **Déboguer plusieurs microcontrôleurs simultanément** dans une seule et même fenêtre STM32CubeIDE.
- **Exécuter le code pas à pas et atteindre des points d'arrêt (breakpoints)** sur différentes cibles en temps réel.
- **Analyser les états et les conditions de concurrence (race conditions)** sans échanger les câbles USB ni deviner l'état de l'autre carte.

---

## Guides de Documentation

* 📘 **[Tutoriel Étape par Étape](TUTORIAL_MULTI_DEBUG.md)** : Trouver les numéros de série des sondes ST-LINK, configurer les ports GDB et créer un "Groupe de Lancement" (Launch Group) pour un démarrage en un clic.
* ❓ **[FAQ & Dépannage](FAQ_TROUBLESHOOTING.md)** : Solutions pour les conflits de ports (erreurs `bind`), les limites d'alimentation USB et le partage des fichiers de configuration via Git.

---

## Principaux Avantages
* **Efficacité :** Réduit considérablement le temps passé à déboguer les communications multi-MCU.
* **Précision :** Synchronisation de l'exécution pas à pas et des points d'arrêt sur différentes cibles.
* **Coût Zéro :** Utilise nos sondes ST-LINK/V2 existantes et les fonctionnalités standards de STM32CubeIDE.
