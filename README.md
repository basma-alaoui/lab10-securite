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
