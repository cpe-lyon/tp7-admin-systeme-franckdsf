# TP 7 - Boot, services et processus / Tâches d’administration (2)

## Exercice 1. Personnalisation de GRUB

**1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il est présent dans votre environnement (vous pouvez aussi commenter son contenu).**

mv /etc/default/grub.d/50-curtin-settings.cfg /etc/default/grub.d/50-curtin-settings.old


**2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’affiche pendant 10 secondes ; passé ce délai, le premier OS du menu doit être lancé automatiquement.**

GRUB_TIMEOUT=10 ; GRUB_TIMEOUT_STYLE=menu 

**3. Lancez la commande update-grub**

J'ai fait `update-grub` 

**4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte**

fait

**5. On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres à rajouter au fichier grub.**

GRUB_GFXMODE=1280x1024,1024x768x32 GRUB_GFXPAYLOAD_LINUX=keep

**6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : grub2-splash-images
(après installation, celles-ci sont disponibles dans /usr/share/images/grub).**

GRUB_BACKGROUND="/home/image1".

**7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*).
Installez-en un**

fait 

**8.  Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer**

/etc/grub.d/40_custom 
```
menuentry ’Arrêt du système’ {
halt
}
menuentry ’Redémarrage du système’ {
reboot
}
```

**9. Configurer GRUB pour que le clavier soit en français**

j'ai fait `mkdir /boot/grub/layouts` puis `grub-kbdcomp -o /boot/grub/layouts/fr.gkb fr` 
j'ai fait dans /etc/default/grub `GRUB_TERMINAL_INPUT=at_keyboard` 
j'ai fait dans /etc/grub.d/40_custom 
```
insmod keylayouts
keymap fr
``` 
## Exercice 2. Noyau


**1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).**

j'ai fait la commande `apt install build-essential` 

**2. Créez un fichier hello.c contenant le code suivant**

j'ai fait `touch hello.c` 

**3.  Créez également un fichier Makefile**

obj-m += hello.o

 all:
 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

 clean:
 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

 install:
 cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc


**4. Compilez le module à l’aide de la commande make, puis installez-le à l’aide de la commande make
install.**


**5. Chargez le module ; vérifiez dans le journal du noyau que le message ”La fonction init_module() est appelée” a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod**

j'ai fait `modprobe -a hello.ko` 

**6. Utilisez la commande modinfo pour obtenir des informations sur le module hello.ko ; vous devriez notamment voir les informations figurant dans le fichier C.**



**7. Déchargez le module ; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module() est appelée” a bien été inscrit, synonyme que le module a été déchargé ; confirmez avec la commande lsmod.**

j'ai fait `modprobe -r hello.ko`  

**8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.**

Je vois le module hello.ko avec lsmod 

## Exercice 3. Interception de signaux


**1. . Commencez par écrire un script qui recopie dans un fichier tmp.txt chaque ligne saisie au clavier par
l’utilisateur**

j'ai fait 
``` 
#!/bin/bash
while :; do
read a
echo $a >> tmp.txt
done
``` 

**2. Lancez votre script et appuyez sur CTRL+Z. Que se passe-t-il ? Comment faire pour que le script poursuive son exécution ?**

la commande `CTRL+Z` met le sript en pause. <br>
j'ai fait `fg k` pour que ça reprenne 

**3. Toujours pendant l’exécution du script, appuyez sur CTRL+C. Que se passe-t-il ?**

Il s'est arreté 

**4. Modifiez votre script pour redéfinir les actions à effectuer quand le script reçoit les signaux SIGTSTP (= CTRL+Z) et SIGINT (=CTRL+C) : dans le premier cas, il doit afficher ”Impossible de me placer en arrière-plan”, et dans le second cas, il doit afficher  "OK, je fais un peu de ménage avant" avant de supprimer le fichier temporaire et terminer le script.**
j'ai fait 
``` 
trap 'echo impossible de me placer en arrière-plan !' TSTP
trap 'echo OK, je fais juste un peu de ménage avant ; exit' INT
trap 'rm tmp.txt' EXIT
``` 

**5.Testez le nouveau comportement de votre script en utilisant d’une part les raccourcis clavier, d’autre
part la commande kill**

**6. Relancez votre script et faites immédiatement un CTRL+C : vous obtenez un message d’erreur vous indiquant que le fichier tmp.txt n’existe pas. A l’aide de la commande interne test, corrigez votre script pour que ce message n’apparaisse plus.**

## Exercice 4. Surveillance de l’activité du système

**1. Dans tty1, lancez la commande htop, puis tapez la commande w dans tty2. Qu’affiche cette commande ?**

elle affiche les utilisateurs connecté, leurs ip si ils sont connecté a distance et leurs processus en cours 

**2. Comment afficher l’historique des dernières connexions à la machine ?**

il faut faire la commande `last` 

**3. Quelle commande permet d’obtenir la version du noyau ?**

il faut faire la commande `uname -r` 

**4. Comment récupérer toutes les informations sur le processeur, au format JSON ?**

il faut faire la commande `lshw -class Processor -json`  

**5. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?
Comment afficher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot ?**

il faut faire la commande 

```
journalctl --list-boots
journalctl -b -1
``` 


**6. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?**

journalctl --list-boots


**7. Faites en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message
à l’écran d’une maintenance le 26 mars à minuit.**

j'ai fait `nano /etc/motd` et j'ai écris a l'intérieur maintenance prévue le 26 mars a minuit 

**8. Ecrivez un script bash qui permet de calculer le k-ième nombre de Fibonacci : Fk = Fk−1 + Fk−2, avec F0 = F1 = 1. Lancez le calcul de F100 puis lancez la commande tload depuis un autre terminal virtuel. Que constatez-vous ? Interrompez ensuite le calcul avec CTRL+C et observez la conséquence sur l’affichage de tload.**
