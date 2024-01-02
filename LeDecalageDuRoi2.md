
# Énoncé :

Bien joué, jeune padawan, maintenant que tu comprends le fonctionnement de ce chiffrement tu as tous les outils nécessaires pour aider la république à déchiffrer les textes que s'envoient les vaisseaux de l'empire galactique. Nous avons réussi à intercepter un flux de connexion que l'empire utilise pour transmettre des informations chiffrées. Le problème, la clé est aléatoire à chaque connexion, seul l'empire les détient, de plus, les connexions ne durent que quelques secondes. Ils ont l'air de se transmettre des textes assez longs, peut-être que cela pourrait constituer une faiblesse dans ce chiffrement... Vous pourrez vous aider des fichiers que l'on a pu récupéré.

PS : Le texte en clair est en français

connexion : `nc sockets.challs.flagmalo.fr 2222`
  

# Explication :

OK, donc lorsqu'on se connecte au socket on tombe sur un "Bienvenue dans ce challenge de Caesar...", on connaît donc le chiffrement, lorsqu'on appuie sur 1, un texte chiffré nous est donné et on doit rentrer sa clé. Au bout de trois secondes, le flux se ferme.

Il y a deux fichiers qui nous sont donnés pour résoudre ce challenge, le fichier qui tourne sur le serveur (avec le flag enlevé bien sûr) `server_caesar.py` et un fichier `vigenere.py`
Ce dernier est juste un fichier qui permet de déchiffrer du Vigénère et donc du César.

Fonctionnement du chiffrement de Vigénère : 
C'est un chiffrement par décalage, comme César, par contre, la différence c'est que la clé est représentée sous forme de mots composées des lettres de l'alphabet romain, et le décalage du texte en clair varie en fonction de la lettre de la clé.
la lettre a constitue un décalage de 0 (la lettre du texte en clair ne change pas)
Pour b ce sera 1....jusqu'à z qui représentera un décalage de 25.
Voici une représentation de ce chiffrement
![vigenere representation](https://img.geocaching.com/cache/large/12fb2b9f-387a-4dc1-a20c-45903c30bdd3.jpeg)

La clé n'a bien sûr pas besoin d'avoir la même taille que le texte en clair, elle peut-être plus courte, et dans ce cas lorsqu'elle finit le chiffrement du reste du texte en clair va recommencer avec le début de la clé :
texte = defghi 
clé = abc
texte chiffré = dfhgik

et si la clé ne fait qu'un seul caractère on obtient alors le chiffrement de César.

Je ne vais pas rentrer dans les détails du code pour chiffrer et déchiffrer en vigénère car ce n'est pas le but de ce chall. Si vous comprenez son fonctionnement c'est l'essentiel.

Maintenant regardons ce qui se passe du côté du serveur : 

    .
    .
    def  shufflewords(line):
	    words = line.split()
	    shuffle(words)
	    return  '  '.join(words)+"\n"
    def  generatekey():
	    alpha="bcdefghijklmnopqrstuvwxyz"
	    return  choice(alpha)
	 .
	 .
	 .
	 if  int(num)  ==  1:
	    s.send(b"voici le texte que vous devez dechiffrer :\n")
	    time.sleep(0.5)
	    encfile =  open("cryptcaesar.txt",  "wb")
	    crypted =  []
	    place =  ""
	    with  open("sample.txt",  "rb")  as text:
		    bytelines = text.readlines()
		    strlines =  [shufflewords(x.decode()[:-1])  for x in bytelines]
		    key =  generatekey()
		    place =  vigenere("".join(strlines).lower(), key)
		    crypted.append(''.join(place))
		    for encline in crypted:
			    encfile.write(bytes(encline,  encoding="UTF-8"))
			    s.send(encline.encode())
	    s.send(b"\nRentrez la clef : ")
        if  read_line(s).decode()  == key:
		    s.send(b"Bravo! Voici votre flag: FMCTF{F4ke_FlaG}")
	    else  :
		    s.send(b"skill issue")
Vous  remarquerez que je n'ai pris que ce qui nous intéresse.

La fonction `shufflewords` désordonne tous les mots dans une phrase.
La fonction `generatekey` retourne une lettre aléatoire (sauf a!!).

Maintenant, losque l'on appuie 1 pour accéder au texte chiffré, le serveur ouvre un fichier texte mélange les mots des lignes (comme ça le texte en clair change à chaque fois), génère la clé, chiffre le texte en César (la clé ne fait qu'un caractère comme dit précédemment) et l'envoie au client en demandant la clé. Si le client la trouve, le serveur lui donne le flag.

# Résolution :
Selon l'énoncé, le fait que le texte est long pourrait être une faiblesse pour le chiffrement de César. En effet, l'analyse de fréquence marcherait très bien dans cette situation.

Qu'est-ce-que l'analyse de fréquence : 
La lettre la plus fréquente dans la langue française est "e", donc naturellement tous les textes (livres, documents etc) auront "e", comme lettre la plus fréquente.
Supposons qu'un texte parreille est chiffré en César avec b comme clé, logiquement, la lettre la plus fréquente ne sera plus "e" mais "f", si on retrouve cette dernière lettre alors on retrouve la clé "b".

C'est cette technique que nous allons utiliser pour ce challenge. 
Grâce à un code python, nous nous connectons d'abord au serveur, on envoit "1\n" pour recevoir le texte chiffré et on compte la lettre la plus présente dans ce texte, lorsqu'on la trouve, on la déchiffre à l'aide de la fonction `decrypt_vigenere` (et oui, c'est surtout à ça qu'elle servait! ) qui nous est fournie, avec comme paramètre ([Lettre la plus fréquente dans le texte chiffré], e). On envoit la clé, et on reçoit notre flag ! 

    Flag : FMCTF{C43saR-iS_LaM3-Us3_V1GeN3rE_InSt34D}

Voici une proposition de code qui pourrait servir pour résoudre ce challenge : 

    import socket
    from collections import Counter
    from vigenere import decrypt_vigenere
    
    #hostname et port du serveur distant
    ip =  'sockets.challs.flagmalo.fr'
    port =  2222
    
    #Création du socket
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    rcvd_data_log =  []
    try:
    	#Connexion au serveur
    	client_socket.connect((ip, port))
    	print(f"Connecté au serveur {ip}:{port}")
    	while  True:
    		#Recevoir les données du serveur
    		rcvd_data = client_socket.recv(4096) #taille du buffer : 4096
    		rcvd_data_log.append(rcvd_data)
    		if  not rcvd_data:
    			break
    		#Envoyer des données au serveur
    		if  "votre choix"  in rcvd_data.decode('utf-8'):
    			client_socket.sendall(b"1\n")
    		elif  "Rentrez la clef"  in rcvd_data.decode('utf-8'):
    			spaced_ciphertext="".join(rcvd_data_log[-2].decode('utf-8')[:-1])
    			ciphertext="".join(spaced_ciphertext.split())
    			res =  Counter(ciphertext)
    			res =  max(res,  key  = res.get)
    			encrypted_key = res
    			#Comme la lettre la plus commune pour chaque morceau de texte est e
    			key =  decrypt_vigenere(encrypted_key,"e")
    			client_socket.sendall(key.encode()+b"\n")
    			rcvd_data = client_socket.recv(4096)
    			print("Données reçues du serveur:", rcvd_data.decode('utf-8'))
    		else:
    			continue
    except  ConnectionRefusedError:
    	print("La connexion a été refusée. Vérifiez l'adresse IP et le port du serveur.")
    finally:
	    #Fermer la connexion
	    client_socket.close()

À noter que comme la clé ne fait qu'un caractère, il est possible de ne tester qu'un seul caractère à chaque connexion et au bout d'un moment la clé sera vraiment la bonne.
Parcontre si le challenger utilise cette méthode, il aura définitivement des difficultés pour résoudre le challenge suivant...

# Difficulté Moyenne:
Ce challenge est de difficulté moyenne car la notion d'analyse de fréquence n'est pas connue de tous. De plus la réalisation d'un code python pour communiquer avec un serveur n'est pas si simple, surtout si l'on n'est pas familié avec le langage.


