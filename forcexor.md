
# Énoncé :

Hmm magique, est la force...Nos données, l'Empire a chiffré avec un ransomware. À récupérer ma photo préférée, aide-moi, tu dois.

  

# Explication :

Donc, dans ce challenge, nous avons un fichier python, `forcexor.py` et un fichier chiffré, `flag.enc`, regardons le code python fournit de plus près :

    import os  
    from itertools import cycle  
      
    key = os.urandom(12)  
      
    filedata =open("FlagMagic.jpg","rb").read()  
    encfile=open("flag.enc","wb")  
      
    for data,keyy in zip(filedata,cycle(key)):  
    encfile.write(bytes([data^keyy]))  
    os.remove("FlagMagic.jpg")

En bref, `os.urandom(12)`, génère une suite de 12 octets aléatoires qu’on stocke dans la variable `key`.

On ouvre le fichier `FlagMagic.jpg`, en lecture binaire qui contient le flag en clair et on ouvre ensuite un nouveau fichier flag.enc en écriture binaire.

Ensuite, la partie la plus importante, on xor chaque octet de la clé avec les octets du fichier.

Vous pourriez penser que la clé est trop courte pour chiffrer tout le fichier jpg, et vous avez raison. C’est là où intervient la fonction `cycle` qui va faire une boucle de la clé sur les octets de l’image, c’est-à-dire que dès que la clé est terminée, elle recommence à chiffrer à partir de son début, et ce, jusqu’à la fin du fichier.

Et enfin, on supprime l’image. Cela pourrait ressembler au fonctionnement d’un ransomware (ou rançongiciel pour les plus français).

  

# Résolution :

Bon maintenant, comment on déchiffre le fichier???

Il y a plusieurs axes de réflexion qu’un challenger pourrait avoir :

 1. bruteforce 12 octets :
    
    12 octets qu’est-ce-que ça représente, 96 bits, combien de nombres
    peut-on avoir avec 96 bits, 296 ou 7.9228163e+28, autant vous dire
    que vous passerez une éternité avant de trouver la bonne clé.

  

 2. Se pencher sur les propriétés du XOR :
    
    on est en binaire :
    
    Soit a = 1101 et b=1001, tel que a ⊕ b = c (⊕ étant l’opérateur xor)
    
    c sera alors égal 0100.
    
    a ⊕ c = 1001 = b, on peut en déduire que si l’on connaît deux des
    variables a, b ou c, on peut facilement trouver celui qui reste. 

Appliquons ce principe sur ce challenge :    
nous avons le fichier chiffré que l’on va appeler `enc`, donc on doit chercher soit la clé de douze octets (et comme dit précédemment ça va prendre trop de temps) ou soit on peut utiliser le fichier en clair. Sauf qu’on ne l’a pas…   En réalité, il y a quelque chose que
tous les fichiers d’un même format ont en commun et on les appelle les `magic numbers`, ce sont des octets signatures par lesquels tous les fichiers d'un même format commencent. Il s’avère que pour les fichiers jpg, ils font exactement douze octets!

Si on part sur [wikipédia](https://en.wikipedia.org/wiki/List_of_file_signatures) on peut en voir une liste.

Pour jpg : FF D8 FF E0 00 10 4A 46 49 46 00 01

Donc si on prend, les 12 premiers octets du fichier chiffré, et les douze octets des magic numbers qu’on a trouvé, on retrouve notre clé ! Autrement dit :

enc ⊕ jpg = clé.

et avec cette clé on peut utiliser le programme fourni pour qu’il fasse l’inverse, et on retrouve l’image en clair :

![](https://lh7-us.googleusercontent.com/BJgaNXB5oqrgkeeIEHo-eFKzR6ulrBrigXJBgjXstp0x6s2P5jxs1GIGWraQVXdpuWvOHObO6_dITj2G8COjhBzn32urvgtndO2qQ2WKDzmGKBfgYEyGkeOdo9oyTaWVD0m1n66jITf4PjqZk5kgpNY)

Voici une proposition de code qui sert à avoir le flag :

    from itertools import cycle  
    from pwn import xor  
    encrypted = open("flag.enc", "rb")  
    encrypted_magicbytes = encrypted.read(12)  
      
    key = xor(encrypted_magicbytes,b'\xff\xd8\xff\xe0\x00\x10JFIF\x00\x01')  
      
    data = open("flag.enc","rb").read()  
    out = open("FlagMagic.jpg","wb")  
      
    for d,k in zip(data,cycle(key)):  
    out.write(bytes([d^k]))

  

`b'\xff\xd8\xff\xe0\x00\x10JFIF\x00\x01'` constitue juste les magic numbers du jpg sous format d’octets.  
  

# Difficulté Moyenne:

En réalité j’hésitais à le mettre en difficile, mais finalement je l’ai mis en moyen car le code fourni aide quand même assez à comprendre ce qui se passe.

Par contre, il faudrait déjà être à l’aise en python, connaître les propriétés du xor et les magic numbers ou au moins faire des recherches poussées pour trouver des informations dessus.
