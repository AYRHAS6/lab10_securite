# Frida — Instrumentation Dynamique sur Android avec Windows & Genymotion

## Qu'est-ce que Frida ?

Frida est un framework d'instrumentation dynamique open source, développé par Ole André Vadla Ravnås. Il permet d'injecter du code JavaScript dans des processus natifs en cours d'exécution, que ce soit sur Android, iOS, Windows, macOS ou Linux, sans nécessiter le code source de l'application cible.

Concrètement, Frida permet d'intercepter des appels de fonctions, de modifier le comportement d'une application à la volée, d'analyser des échanges réseau chiffrés, ou encore de contourner des mécanismes de protection comme la détection de root ou le certificate pinning. C'est un outil incontournable en reverse engineering, en pentest mobile et en analyse de malware.

---

## Architecture : Client / Serveur

Frida repose sur une architecture **client/serveur** composée de deux éléments distincts qui communiquent ensemble.

### frida-server (côté émulateur / appareil Android)

`frida-server` est un binaire qui s'exécute directement sur l'appareil Android cible — ici l'émulateur Genymotion. Il tourne en arrière-plan avec les droits root et expose une interface permettant au client de s'y connecter. C'est lui qui réalise concrètement l'injection de code dans les processus Android.

Dans ce projet, `frida-server` est déposé sur l'émulateur via ADB et lancé manuellement avant chaque session d'analyse :

<img width="1528" height="118" alt="1" src="https://github.com/user-attachments/assets/a2ff6df9-513e-40b4-8480-460f587b04bc" />
<img width="1069" height="80" alt="2" src="https://github.com/user-attachments/assets/3560ee3c-26cb-4a18-9d73-c88c29000d71" />




>  La version de `frida-server` doit correspondre exactement à la version du client installée sur Windows.

### frida-client (côté Windows)

Le client Frida s'installe sur la machine de l'analyste — ici Windows — via pip. Il regroupe la bibliothèque Python `frida` ainsi que les outils en ligne de commande `frida-tools` (`frida`, `frida-ps`, `frida-trace`, etc.).

```powershell
pip install frida-tools
```
<img width="370" height="58" alt="3" src="https://github.com/user-attachments/assets/f5cf1b0b-ed5e-4e05-b145-ee40fb397441" />


C'est depuis le client que l'analyste écrit et envoie les scripts JavaScript vers le processus cible, liste les processus en cours, ou s'attache à une application spécifique.

### Communication

Le client Windows se connecte à `frida-server` sur l'émulateur via la connexion ADB (USB ou réseau local). Dans le cas de Genymotion, la connexion se fait par réseau local sur l'IP de l'émulateur (`192.168.56.102:5555`). L'option `-U` des commandes Frida désigne cette connexion.

```
Windows (frida-client)  <──ADB/réseau──>  Genymotion (frida-server)  <──injection──>  Application Android
```

---

## Environnement utilisé

| Composant | Détail |
|-----------|--------|
| OS analyste | Windows 11 |
| Client Frida | frida-tools 14.8.1 / frida 17.9.1 |
| Émulateur | Genymotion — Pixel 3 (Android x86) |
| Serveur Frida | frida-server-17.9.1-android-x86 |
| Connexion ADB | 192.168.56.102:5555 |
| Python | 3.13 |

---

## Validation de l'installation

Une fois `frida-server` démarré sur l'émulateur, la commande suivante permet de lister les processus Android depuis Windows :

```powershell
frida-ps -U
```
<img width="598" height="621" alt="4" src="https://github.com/user-attachments/assets/c00193c3-165f-4fd1-a5cf-e6147affba46" />


Un premier test d'injection avec l'API Java de Frida confirme que la chaîne complète fonctionne :

```javascript
// hello.js
Java.perform(function () {
  console.log("[+] Frida Java.perform OK");
});
```

```powershell
frida -U -f com.android.calculator2 -l hello.js
```

Résultat attendu dans la console Frida :

```
[+] Frida Java.perform OK
```
<img width="1080" height="363" alt="5" src="https://github.com/user-attachments/assets/d1012b4b-0ade-464c-875f-e9fd3080fbb3" />

Ce résultat confirme que le client Windows est bien connecté à l'émulateur, que `frida-server` est opérationnel, et que Frida peut injecter et exécuter du code JavaScript dans un processus Android. L'application Calculatrice s'est ouverte sur l'émulateur Genymotion et le message `[+] Frida Java.perform OK` confirme que Frida a bien injecté et exécuté le script JavaScript dans le processus Android.
<img width="589" height="1027" alt="6" src="https://github.com/user-attachments/assets/379f8459-e0b8-4a42-9a84-5864ae294b9c" />


