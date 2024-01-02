
# Énoncé :

ETAPE 4 : Votre détermination nous rapproche chaque fois davantage de la victoire. Cependant, une nouvelle épreuve se dresse devant vous. Les plans que vous avez récupérés sont protégés par un puissant chiffrement. Le temps presse, et la République compte sur vous pour déchiffrer ces informations cruciales avant qu'il ne soit trop tard.

# Explication : 
(Si vous ne connaissez pas le fonctionnement du chiffrement RSA je vous conseille d'essayer les cours de maths et de chiffrement asymétrique sur [CryptoHack](https://cryptohack.org/))
Regardons le code qui nous est fourni : 

    from Crypto.Util.number import getPrime,bytes_to_long, long_to_bytes 
    from itertools import cycle
    
    p= getPrime(1024)
    q= getPrime(1024)
    
    N=p*q
    
    e=2
    
    plainkey = ???
    
    enckey = pow(bytes_to_long(plainkey), e, N)
    
    print(f"N = {N}")
    print(f"exposant = {e}")
    print(f"clef_chiffre = {enckey}")
    
    filedata =open("death-star-plans.pdf","rb").read()
    encfile=open("death-star-plans.enc","wb")
    
    for data,keyy in zip(filedata,cycle(plainkey)):
      encfile.write(bytes([data^keyy]))


Ce code xor tous les octets d'une image avec une clé, mais tout ce que l'on a c'est la clé chiffré, et les paramètres public du RSA.

# Résolution :
L'exposant e qui est égal à 2...
Toute personne qui connaît le RSA, se dit que la faille se situe là-bas. Et c'est exact. 
Comment retrouver la clé privé d avec un petit e? La vérité c'est que dans ce challenge, on n'en a même pas besoin.
Prenons un exemple : 
12² ≡ 144 [100] => 12² ≡ 44 [100]
Dans ce cas si on a que la deuxième congruence et qu'on ne connait pas le 12², c'est-à-dire x ≡ 44 [100], difficile de retrouver x ! 
12² ≡ 144 [169]
Dans ce cas...La congruence reste comme ça et si qu'on a que  : x² ≡ 144 [169], on retrouve notre x en  faisant tout simplement la racine carrée de 144, donc 12.
Vous voyez où je veux en venir?
Il suffit juste de faire : 
key =  √enc_key, on retrouve notre clé :

    ------V0ic1__-_*$L4__Cl3f_QuE___VoU5__-___Ch3rChEz------

et on déchiffre le fichier : 
![](https://lh7-us.googleusercontent.com/J6AV6504FhTFCixgqVyriPvRJL9PBOnlbXdCNtFH5q_Nqyzp48noHxMFQhMpEVxNccdlmsTT5Xs7sZng1E2D5Gk9IfL6JKGe8wcRB0-y9QCnBG_G2wq3CBku9_KBhVg-3sbZ6XQKBZVvOc6cGUMkdLU)
# Difficulté Difficile : 
Le RSA requiert une base mathématique en arithmétique assez forte, ce qui n'est pas le cas de beaucoup d'étudiants en première ou en deuxième année d'université. De plus la compréhension du code python est essentielle pour résoudre ce challenge. C'est pour cela qu'il est difficile.
