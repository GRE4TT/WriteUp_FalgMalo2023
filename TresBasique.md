
# Énoncé :
Arriverez vous à déchiffrer ce message ? 
39 44 21 37 41 3K 2F 4E 2J 3K 31 1N 39 4F 3K 34 42 3K 2G 22 4F 2J 4F

indice : 
10 : 0123456789 
11 : 0123456789A
12 : 0123456789AB
...
16 : 0123456789ABCDEF
...
25 : 0123456789AB...MNO

Format du Flag : FMCTF{ce_que_vous_trouverez}

PS : Les mots de deux caractères sont séparés pour une raison

# Résolution :
Vu le nom du challenge et l'indice qui nous a été donné, on pourrait se dire que le texte a été codé dans une base. Mais laquelle?
J'aurais pu aller jusqu'à la base 36 qui va de 0 à 9 et de A à Z, mais je me suis arrêté à 25 pour une raison...
On pourrait alors se dire que le message est codé en base 25.
Décodons le alors, on peut aller sur [dcode](https://www.dcode.fr/base-n-convert) qui a un outil pour convertir n'importe quelle base en une autre, on peut alors convertir notre message de base 25 en base 16 (hexadecimal), en faisant bien attention à convertir chaque mot de deux caractères séparément car `39 44` en base 25 donne `54 68` en hexadécimal, alors `3944` donne `CD7C`. 
Bref on obtient en hexadécimal : 

    54 68 33 52 65 5F 41 72 45 5F 4C 30 54 73 5F 4F 66 5F 42 34 73 45 73

puis on le convertit grâce à la table ASCII ([dcode](https://www.dcode.fr/ascii-code)) et on obtient : 

    Th3Re_ArE_L0Ts_Of_B4sEs

# Difficulté Facile : 
Bon, c'est vrai que le cheminement de pensée, pour arriver à trouver la base 25 à la convertir dans une autre base puis en ASCII n'est peut-être pas si évidente. Mais il n'y a que deux étapes pour résoudre ce challenge, c'est pour cela qu'il est "Facile".
