
# Énoncé :

L'étoile de la mort veut transmettre des informations importantes et on a intercepté leur message, Grâce à vos connaissances dans les chiffrements asymétriques, aidez nous à le déchiffrer

# Explication : 
(Si vous ne connaissez pas le fonctionnement du chiffrement RSA je vous conseille d'essayer les cours de maths et de chiffrement asymétrique sur [CryptoHack](https://cryptohack.org/))
Regardons le code qui nous est fourni : 

    from Crypto.Util.number import getPrime, bytes_to_long, inverse

    def encrypt_flag(flag):
        p = getPrime(1024)
        q = getPrime(1024)
        N = p*q
        # get Euler's totient, we'll call it phi(N)
        phi_N = (p-1)*(q-1)
        cipherlist = []
        exponants = []
        x = 10
        for i in range(0, len(flag), 6):
            flagpiece = flag[i:i+6].decode('utf-8')
            d = getPrime(x)
            e = inverse(d, phi_N)
            exponants.append(e)
            # A long is a large integer
            flagtolong = bytes_to_long(flagpiece.encode('utf-8'))
            cipherpiece = pow(flagtolong, e, N)
            cipherlist.append(cipherpiece)
            if x < 625:
                x *= 5
        return N, cipherlist, exponants
    
    
    def main():
        #La taille du flag n'est pas respectée dans ce code
        flag = b"FMCTF{???????????}"
        N, cipherlist, exponants = encrypt_flag(flag)
        print(f"N = {N}")
        print(f"exponants = {exponants}")
        print(f"cipherlist = {cipherlist}")    

    if __name__ == '__main__':
        main()
La fonction encrypt_flag, génère deux nombres premiers de 1024 bits stockées dans les variables p et q.
On génère le modulant du RSA N, en multipliant p et q et on récupère le totient d'Euler  phi_N.
C'est maintenant où le chiffrement diverge un peu du RSA-2048 basique : 
on déclare et instancie une variable x ayant une valeur de 10.
Puis pour chaque 6 caractères du flag, on génère d, qui est la clé privé (exposant privé) en prenant un nombre premier de x bits (10 dans ce premier cas).
On génère e (clé ou exposant public), qui est l'inverse modulaire de d modulo phi_N.
On chiffre ensuite les 6 caractères du flag pris précédemment, et tant que x n'est pas supérieur à 625, on multiplie x par 5.
À la fin N, la liste des exposants publics et des morceaux de 6 caractères de flag chiffré nous sont donnés

# Résolution :
La première erreur qui saute aux yeux et la première clé privée d qui ne fait que 10 bits.
10 bits, c'est **très** facile à bruteforce, on peut alors trouver la première clé privé d, en sachant que les 6 premiers caractères du flag sont FMCTF{.
Ensuite, on peut retrouver le totient d'Euler phi_N (qui pour rappel reste le même pour tous les morceaux chiffrés du flag) qui n'est qu'un diviseur de ed-1 (car ed ≡ 1 [phi_N]).
Avec phi_N on peut retrouver toutes les autres clés privées même si ce sont des nombres colossaux.
Voici une proposition de code pour résoudre ce challenge : 

    from Crypto.Util.number import getPrime, long_to_bytes
    
    N =  #Prendre la valeur qui est dans le fichier output
    exponants = #Prendre la valeur qui est dans le fichier output
    cipherlist = #Prendre la valeur qui est dans le fichier output
    
    ciphertext = cipherlist[0]
    e0 = exponants[0]
    
    while True:
        d0 = getPrime(10)
        plaintext = pow(ciphertext, d0, N)
        if long_to_bytes(plaintext) == b"FMCTF{":
            break
    
    #e*d = 1 (mod phiN) => (e*d)-1 = 0 (mod phiN) => (e*d)-1 divise phiN
    #Dans tous les cas (Tout ceux que j'ai testé) (e*d)-1 = phiN, alors... 
    phiN = e0*d0-1
    
    flag = b""
    for i in range(len(cipherlist)):
        d = pow(exponants[i], -1, phiN)
        flagpiece = long_to_bytes(pow(cipherlist[i], d, N))
        flag += flagpiece
    
    print(flag)

Flag : `FMCTF{4_5mALL_Pr1M3_C4n_Br34K_3veRyTh1Ng_}`

# Difficulté Difficile : 
Le RSA requiert une base mathématique en arithmétique assez forte, ce qui n'est pas le cas de beaucoup d'étudiants en première ou en deuxième année d'université. De plus la compréhension du code python est essentielle pour résoudre ce challenge. C'est pour cela qu'il est difficile.
