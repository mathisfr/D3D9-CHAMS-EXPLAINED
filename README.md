# Création d'un Chams Hack pour DirectX9  
> Il est important de noter que ces manipulations sont présentées uniquement à des fins éducatives et de compréhension des principes sous-jacents du développement de jeux.
![cod4chamshack](https://github.com/mathisfr/D3D9-CHAMS-EXPLAINED/blob/main/cod4chams.png)
* **Introduction**
Bienvenue dans ce tutoriel dédié à l'explication du fonctionnement d'un chams hack pour DirectX9.  
Les chams hacks sont des modifications de rendus graphiques dans les jeux vidéo qui permettent de rendre les modèles des joueurs plus visibles, facilitant ainsi leur détection.  
Ce guide est spécialement conçu pour les développeurs et les passionnés de cybersécurité souhaitant explorer les techniques de hacking éthique dans le cadre de DirectX9.  

L'objectif de ce tutoriel est de vous fournir une compréhension complète des étapes nécessaires pour créer un chams hack pour DirectX9.  

> J'avais initialement prévu de partager tous mes fichiers après avoir créé un chams hack pour COD4.  
> Cependant, une erreur s'est produite lors du push forcé de mes fichiers, ce qui a malheureusement conduit à la suppression de tous mes fichiers d'en-tête en raison d'une mauvaise manipulation. 😅  
> Par conséquent, j'ai décidé d'écrire un tutoriel à la place. ✍️   

* **Prérequis**

    ***Connaissances Techniques:***  
        - Compréhension de base de DirectX9  
        *(Si vous n'avez jamais utilisé DirectX 9, je vous conseille vivement de commencer par les premières parties de ce tutoriel pour bien comprendre la suite  : https://www.braynzarsoft.net/viewtutorial/q16390-01-dx9-a-little-about-directx)*  
        - Connaissances en programmation C++  
        - Connaissances en reverse engineering  
    ***Outils Nécessaires:***  
        - Environnement de développement (Visual Studio recommandé)  
        - SDK DirectX9

## Section 1: La base
Pour commencer nous allons avoir besoin d'un plan.  
La méthode que je vais utiliser dans ce tutoriel est DrawIndexedPrimitive de DirectX 9.   
Cette méthode permet de dessiner des primitives (triangles, lignes, points) en utilisant des indices, ce qui est particulièrement utile pour réutiliser des sommets et optimiser ainsi la mémoire et les performances.   
Cette méthode est largement utilisée dans de nombreux jeux DirectX 9.   

La méthode **DrawIndexedPrimitive** prend plusieurs paramètres qui définissent la manière dont les primitives doivent être dessinées :

1. Type de Primitive (D3DPRIMITIVETYPE) : Indique le type de primitives à dessiner (par exemple, D3DPT_TRIANGLELIST, D3DPT_TRIANGLESTRIP, D3DPT_LINELIST, etc.).
2. BaseVertexIndex (INT) : Valeur ajoutée à chaque index avant d'accéder aux données de sommet. Cela permet d'utiliser des sommets à partir d'un sous-ensemble spécifique du vertex buffer.
3. MinIndex (UINT) : Index minimum dans le tableau d'indices. Utilisé pour optimiser le rendu en limitant les vérifications.
4. NumVertices (UINT) : Nombre de sommets à traiter. Utilisé avec MinIndex pour déterminer la plage de sommets à transformer.
5. StartIndex (UINT) : Index de départ dans le tableau d'indices pour commencer à dessiner.
6. PrimitiveCount (UINT) : Nombre de primitives à dessiner.

L'objectif est donc de détourner cette fonction afin de l'exploiter à notre avantage, étant donné qu'elle est appelée pour dessiner les objets à l'écran.  

## Section 2: Hook de la methodes
La méthode que je vais utiliser dans ce tutoriel est comme je l'ai dit une méthode.  
Cela signifie qu'elle peut uniquement être appelée depuis un objet spécifique.   
Cet objet est **IDirect3DDevice9,** le même objet utilisé pour appeler des méthodes telles que **EndScene** dans l'application.  
Ces informations sont essentielles pour contourner le programme.  
Pour cela, je propose d'utiliser le **VMT hook**, cela implique de remplacer directement la méthode dans la table virtuelle (VMT) d'une classe.  
Vous avez deux options simples : Créer un autre **IDirect3DDevice9** dans votre DLL au moment de l'injection, permettant de partager la même table virtuelle et ainsi de hooker les méthodes, ou bien utiliser un debugger pour trouver un pointeur statique dans l'application vers l'objet **IDirect3DDevice9** et remplacer les méthodes dans votre DLL.  
Voici à quoi pourrait ressembler votre fonction qui encapsule l'original à ce moment :
```cpp
// hk = hook
// og = original

HRESULT APIENTRY hkDrawIndexedPrimitive(LPDIRECT3DDEVICE9 pDevice, D3DPRIMITIVETYPE Type, INT BaseVertexIndex, UINT MinIndex, UINT NumVertices, UINT startIndex, UINT primitiveCount){
    return ogDrawIndexedPrimitive(pDevice, Type, BaseVertexIndex, MinIndex, NumVertices, startIndex, primitiveCount);
}
```

## Section 3: Création du Chams Hack
Une fois que vous avez hooké cette fonction, vous pouvez très simplement mettre en place un chams voire un wallhack.  
Pour cela, il vous suffit de jouer avec les paramètres **NumVertices** et **primitiveCount** afin de repérer les objets du jeu qui vous intéressent.  
Cependant, actuellement, il est difficile de le faire car nous n'avons pas de texture ou autre élément visuel pour vérifier cela dans le jeu.  
Vous pouvez réaliser cela en créant une texture avec **IDirect3DBaseTexture9** et en l'appliquant à l'aide de :
```cpp
pDevice->SetTexture(0, YourTexture);

```
Juste avant de retourner à la méthode originale, vous pouvez ajouter une condition basée sur le nombre de sommets et de primitives, par exemple.
> Vous pouvez consulter ce lien [ici](https://www.mpgh.net/forum/showthread.php?t=185844), il s'avère très utile. Il propose également une fonction pour vous montrer comment créer une texture d'une couleur.

Une astuce que j'ai utilisée consiste à enregistrer une structure dans une Set de tous les types, nombres de sommets (NumVertices) et nombres de primitives (primitiveCount) du jeu. Ensuite, en utilisant les touches du clavier, je peux ajuster la condition en temps réel et voir les objets changer de couleur dans le jeu.

Avec cela, votre chams devrait normalement fonctionner, bien que ce soit une méthode assez basique.  
Par exemple, vous pouvez ajouter un wallhack en désactivant simplement le test de profondeur avec **SetRenderState**.

### Conclusion
En conclusion, nous avons exploré la méthode DrawIndexedPrimitive de DirectX 9, essentielle pour dessiner des primitives en utilisant des indices, ce qui optimise la mémoire et les performances en réutilisant les sommets.  
Nous avons également discuté de l'application du hook VMT pour modifier dynamiquement le comportement de cette méthode dans les jeux DirectX 9.  
En combinant ces concepts avec des techniques avancées telles que le chams et le wallhack, il devient possible de personnaliser l'expérience de jeu en temps réel.  
Bien que complexe, ce processus ouvre la voie à des applications créatives dans le domaine du modding et du développement de jeux.

#### Liens utile
https://www.braynzarsoft.net/viewtutorial/q16390-01-dx9-a-little-about-directx (Learn d3d9)    
https://www.mpgh.net/forum/showthread.php?t=185844 (Very helpful d3d9 things)   
https://www.unknowncheats.me/wiki/Direct3D (More about d3d9 hacking)    
https://github.com/furkankadirguzeloglu/ImGuiHook-DirectX9 (Example and D3D9 Methods Tables)    
https://www.codereversing.com/archives/181 (VMT hook)    

#### Note
Il est important de noter que ces manipulations sont présentées uniquement à des fins éducatives et de compréhension des principes sous-jacents du développement de jeux.  
N'hésitez pas à me contacter ou à laisser un commentaire si vous souhaitez que j'approfondisse davantage ou corrige des erreurs.
