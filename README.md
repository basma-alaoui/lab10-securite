            Installation de Frida sur la machine hôte (PC)

Commandes exécutées:
powershell
# Vérification de Python
python --version
pip --version

# Installation du client Frida
pip install --upgrade frida frida-tools

# Vérifications finales
frida --version
frida-ps --version
python -c "import frida; print(frida.__version__)"
                         
                         Résultat attendu
                         
Les trois versions (CLI frida, frida-ps et module Python) doivent être identiques (ex. 17.9.10).

Pas d’erreur ModuleNotFoundError.

La commande frida-ps doit afficher une liste (vide ou non) des processus sur l’hôte (ou erreur si aucun appareil connecté, ce qui est normal à ce stade).
                       
                       Objectifs du TP:
                       
Installer le client Frida sur le PC (Windows/Linux/macOS)

Déployer frida-server sur un appareil Android (physique ou émulateur)

Vérifier la communication entre PC et appareil

Injecter des scripts JavaScript dans un processus Android

Observer des comportements applicatifs (réseau, stockage, vérifications de sécurité)

                Environnement requis
Composant	Spécification
PC hôte	Windows 10/11, Linux ou macOS
Python	3.10+ (avec pip)
ADB (Android Platform Tools)	Version récente
Frida client	17.x (installé via pip)
Appareil Android	Physique (débogage USB activé) ou émulateur (AVD)
Câble USB	Pour appareil physique (ou connexion Wi-Fi ADB)
7-Zip (Windows)	Pour extraire frida-server.xz
Étape 1 : Installation de Frida sur la machine hôte
1.1 Vérifier Python et pip
Ouvrez un terminal (PowerShell sous Windows, bash sous Linux/macOS) :

bash
python --version
pip --version
Résultat attendu :
Python 3.10.x ou supérieur, pip 22.x ou supérieur.

1.2 Installer le client Frida
bash
pip install --upgrade frida frida-tools
1.3 Vérifier l’installation
bash
frida --version
frida-ps --version
python -c "import frida; print(frida.__version__)"
Résultat attendu :
Les trois commandes affichent la même version (ex. 17.9.1).


                 Étape 2 : Installation et vérification d’ADB
ADB permet la communication avec l’appareil Android.

          2.1 Vérifier ADB
bash
adb version
            
          2.2 Connecter l’appareil / émulateur
Pour un appareil physique : activer « Options développeur » et « Débogage USB ».

Pour un émulateur AVD : il est automatiquement reconnu.

bash
adb devices
Résultat attendu :

text
List of devices attached
emulator-5554   device
ou un identifiant physique XXXXXXXX suivi de device (et non unauthorized).
Si unauthorized, acceptez la clé de débogage sur l’appareil.

               Étape 3 : Déploiement de frida-server sur Android
                3.1 Identifier l’architecture de l’appareil
bash
adb shell getprop ro.product.cpu.abi


             3.2 Télécharger la bonne version de frida-server
Rendez-vous sur Frida Releases – GitHub
Choisissez le fichier correspondant à votre architecture, ex. :
frida-server-17.9.1-android-x86_64.xz (pour x86_64).

            3.3 Extraire l’archive
Sous Windows : utilisez 7-Zip ou tar (si disponible) → tar -xvf frida-server-*.xz

Sous Linux/macOS : xz -d frida-server-*.xz

Renommez le fichier extrait en frida-server (sans extension).

            3.4 Copier frida-server sur l’appareil
bash
adb push frida-server /data/local/tmp/
           
           3.5 Attribuer les droits d’exécution
bash
adb shell chmod 755 /data/local/tmp/frida-server
            
            3.6 Lancer frida-server
Option 1 (visible, pour debug) :

bash
adb shell /data/local/tmp/frida-server -l 0.0.0.0
Option 2 (arrière‑plan, recommandé) :

bash
adb shell "nohup /data/local/tmp/frida-server -l 0.0.0.0 >/dev/null 2>&1 &"
                  
                  3.7 Vérifier que le serveur tourne
bash
adb shell ps | grep frida
Vous devriez voir une ligne contenant frida-server.

                   3.8 Rediriger les ports Frida (si nécessaire)
Par défaut, l’option -U de Frida utilise déjà le forwarding automatique. Mais en cas de problème :

bash
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
                  
                  Étape 4 : Vérification de la connexion PC ↔ appareil
bash
frida-ps -U
ou pour lister les applications installées (avec noms de paquets) :

bash
frida-ps -Uai
Résultat attendu : Une liste de processus / applications Android (ex. com.android.settings, com.android.chrome, etc.). Si la liste est vide, revoyez l’étape 3 (serveur non lancé ou mauvais forwarding).

              Étape 5 : Tests d’injection
              5.1 Script JavaScript simple – hook Java
Créez un fichier hello.js :

javascript
Java.perform(function () {
    console.log("[+] Frida Java.perform OK");
});
Injectez ce script dans l’application Paramètres Android (com.android.settings) :

bash
frida -U -f com.android.settings -l hello.js
Appuyez sur %resume dans le terminal pour laisser l’application démarrer normalement (Frida la suspend au lancement).

Résultat attendu : Dans la console Frida, vous devez voir [+] Frida Java.perform OK après quelques secondes.

                 5.2 Script natif – interception de fonction système (recv)
Créez hello_native.js :

javascript
console.log("[+] Script chargé");

Interceptor.attach(Module.getExportByName(null, "recv"), {
    onEnter(args) {
        console.log("[+] recv appelée");
    }
});
