
# Énoncé :
Oh non! L'empire a changé de flux de connexion. Nous avons réussi à le retrouver, cependant on dirait qu'ils utilisent un chiffrement différent, une variante de celui qu'ils utilisaient précédemment qu'ils appellent Vigenère selon nos sources. L'analyse fréquentielle marchait bien avec le chiffrement de César, il faudra se pencher sur ce "nouveau" chiffrement afin de trouver ces failles.

    Connexion : nc sockets.challs.flagmalo.fr 1111

# Explication :
Ce challenge est comme le dernier mais le serveur chiffre le texte avec une clé de 8 caractères.
(Pour comprendre Vigénère, voir le writeup du challenge précédent)

# Résolution :
En réalité, l'analyse de fréquence marche également sur le chiffrement de Vigénère. Mais c'est assez différent...
Comme le chiffrement change en fonction de la lettre de la clé, la lettre la plus fréquente va également changer en fonction de la lettre.
Prenons ce texte comme exemple : 
"`les hirondelles en ete senvolent vers le ciel bleu et clair celebrant la beaute de la nature avec legerete et gaiete`"
(On enlève les lettres avec accent par la lettre de base comme dans le challenge)
et qu'on la chiffre avec la `clé : oui`
Le texte chiffré résultant est : 

    zya vczchlsftsm mb ybs mmbpwzyvh pmfm ts wqsf jzyc sn kzuqf wmzyjfuvh fi pyiinm ry to hihozs udsw tsamfybs yb uuqsnm

Décomposons ce gros problème, qu'est de déchiffrer tout le texte en vigénère, en une suite de petits problèmes.
Qu'est-ce-que je veux dire par là? 
On va considérer chaque lettre comme une clé de chiffre de César indépendante des autres en récupérant toutes les lettres du texte en clair qui sont chiffrées par chaque lettre de la clé. Pour que cela marche il faudrait connaître la taille de la clé, ce que l'on peut voir dans le code qui tourne sur le serveur.
Lettres du texte chiffrée, chiffrée par la première lettre : 

    zvcssbsbzhfsszszfzfhpirohsssfsus

Lettres du texte chiffrée, chiffrée par la deuxième lettre : 

    ychfmympypmwfynuwyufynyhouwayyun

Lettres du texte chiffrée, chiffrée par la troisième lettre : 

    azltmbmwvmtqjckqmjviimtizdtmbbqm
 
 Et maintenant on déchiffre chaque texte comme des textes chiffrés en César en récupérant les lettres les plus fréquentes : 

    {'char 1': 's', 'char 2': 'y', 'char 3': 'm'}
et on les déchiffre avec "e" comme clé : 

    {'char 1': 'o', 'char 2': 'u', 'char 3': 'i'}

On voit bien qu'on retrouve notre clé avec cette méthode !
Et c'est ce principe qu'on va appliquer sur ce challenge (avec une clé de 8 caractères).

Voici  une proposition de code pour résoudre ce challenge : 

    import socket
    from collections import Counter
    from vigenere import decrypt_vigenere
    #Adresse IP et port du serveur distant
    hostname = 'sockets.challs.flagmalo.fr'
    port = 1111
    
    #hostname = "localhost"
    
    #Création du socket
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    try:
        # Connexion au serveur
        client_socket.connect((hostname, port))
        print(f"Connecté au serveur {hostname}:{port}")
    
        rcvd_data_log=[]
        while True:
            # Recevoir les données du serveur
            rcvd_data = client_socket.recv(8192)
            rcvd_data_log.append(rcvd_data)
    
            if not rcvd_data:
                break 
            # Envoyer des données au serveur
            if "votre choix" in rcvd_data.decode('utf-8'):
              client_socket.sendall(b"1\n")  
            elif "Rentrez la clef" in rcvd_data.decode('utf-8'):
                spaced_ciphertext="".join(rcvd_data_log[-2].decode().split("\n")[:-1])
                ciphertext="".join(spaced_ciphertext.split())
                chardict={}
                charmax={}
                for i in range(8):
                    charlist=[]
                    for j in range(len(ciphertext)):
                        if j%8==i: 
                            charlist.append(ciphertext[j])
                            chardict[f"char {i+1}"]="".join(charlist)
                            res=0
                            res = Counter(chardict[f"char {i+1}"])
                            res = max(res, key = res.get) 
                            charmax[f"char {i+1}"] = res
                encrypted_key = ''.join(charmax.values())
                #Comme la lettre la plus commune pour chaque morceau de texte est e, la clé est une suite de 8
                key = decrypt_vigenere(encrypted_key,"eeeeeeee")
                client_socket.sendall(key.encode()+b"\n")  
                rcvd_data = client_socket.recv(4096) 
                print("Données reçues du serveur:", rcvd_data.decode('utf-8')) 
            else:
                continue
    except ConnectionRefusedError:
        print("La connexion a été refusée. Vérifiez l'adresse IP et le port du serveur.")
    
    finally:
        # Fermer la connexion
        client_socket.close()

 
 Flag : `FMCTF{TH3_0nE_T1me_p4d_iS_tHe_WaY_T0_Go}`

# Difficulté Moyenne-Difficile:
Ce challenge est de difficulté moyenne-difficile. La personne ayant réussi le challenge précédent connait les bases de la connexion client-serveur python et de l'analyse de fréquence sur César. Cependant, étant donné que l'analyse de fréquence sur Vigénère n'est pas évidente, ce challenge est entre moyen et difficile.


