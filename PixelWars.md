
# Énoncé :
La république pense qu'ils pourront se venger du ransomware qu'on leur a envoyé. Ils ont XOR toutes nos images avec une même image clé. Malheureusement, nous n'avons pas pu la récupérer. Aidez l'empire à retrouver les informations de ces fichiers.

# Explication :
Il faut comprendre que ce ne sont pas les octets des images qui ont été XOR, mais plutôt les pixels des images (si le sujet n'était pas assez clair).

# Résolution :
Nous avons deux images pour une raison : 
on sait que ces deux images ont été XOR avec une seule et même image. En connaissant les propriétés du XOR, on peut retrouver nos images en les "XORant". 
Mais comment peut-on faire ça? 
Même si vous n'êtes pas un pro du python vous pouvez trouver un code tout fait sur Google en tapant "XOR images python", c'est pour cela que ce challenge est facile. 
Bon, voici une proposition de code pour résoudre ce challenge : 

    import cv2
    
    #Read two images. The size of both images must be the same.
    img1 = cv2.imread('frogx.jpg')
    img2 = cv2.imread('starx.jpg')
    
    #compute bitwise XOR on both images
    xor_img = cv2.bitwise_xor(img1,img2)
    
    #display the computed bitwise XOR image
    cv2.imshow('Bitwise XOR Image', xor_img)
    cv2.imwrite("flag.jpg",xor_img)
    cv2.waitKey(0)
    cv2.destroyAllWindows()

![](https://lh7-us.googleusercontent.com/rM_b66ig--r86ekF0rDNYbodTDId24X1JaCDPjA8LgZaZhvj9KDjFO4wvsyVi_F3K_qENh_KDDXARYr0Bg70d-964A9OedC8iiuCy82TyRCL9nVi-Mwjr3hEfgnloBdbgX-__NW4TOXog2PPTFJxFuc)

Flag : `FMCTF{n0_1_Am_YoUr_Fr0g3R}`

