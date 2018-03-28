# ES7: async/await dans une boucle

`await` ne peut être utilisé qu'à l'intérieur d'une fonction `async`. C'est un problème lorsqu'on a besoin d'une des fonctions du prototype Array en JS telles que `map` ou `forEach` qui utilisent des callbacks.

## Array.prototype.map

Dans le code suivant, des urls dans un array sont téléchargées et renvoyées dans un autre array.

```js
function async downloadContent (urls) {
    return urls.map(url => {
        // Mauvaise syntaxe
        const content = await httpGet(url);
        return content;
    });
}
```

Cela ne fonctionne pas, car `await` a besoin d'être dans une fonction asynchrone `async`. On pourrait penser qu'il suffit d'ajouter `async` dans la boucle :

```js
function async downloadContent (urls) {
    return urls.map(async (url) => {
        const content = attend httpGet(url);
        return content;
    });
}
``


Il y a deux problèmes avec ce code:

Le résultat est maintenant un tableau de promesses, pas un tableau de string.
Les callbacks ne sont pas achevées une fois que `map()` est terminé, car `await` n'interrompt que sa fonction parente et que `httpGet()` est résolu de manière asynchrone. 

Nous pouvons résoudre les deux problèmes via `Promise.all, qui convertit un tableau de promesses en une promesse pour un tableau (avec les valeurs remplies par les promesses):

```js
fonction async downloadContent (urls) {
    const promiseArray = urls.map(url => httpGet (url));
    return Promise.all (promiseArray);
}
```

## Array.prototype.forEach

Utilisons la méthode Array forEach () pour enregistrer le contenu de plusieurs fichiers pointés via des URL:

Fonction asynchrone logContent (urls) {
    urls.forEach (url => {
        // Mauvaise syntaxe
        const content = attend httpGet (url);
        console.log (contenu);
    });
}
Encore une fois, ce code produira une erreur de syntaxe, car vous ne pouvez pas utiliser await dans les fonctions de flèches normales.

Utilisons une fonction de flèche asynchrone:

Fonction asynchrone logContent (urls) {
    urls.forEach (async url => {
        const content = attend httpGet (url);
        console.log (contenu);
    });
    // Pas fini ici
}
Cela fonctionne, mais il y a une mise en garde: la promesse retournée par httpGet () est résolue de manière asynchrone, ce qui signifie que les rappels ne sont pas terminés lorsque forEach () renvoie. Par conséquent, vous ne pouvez pas attendre la fin de logContent ().

Si ce n'est pas ce que vous voulez, vous pouvez convertir forEach () en boucle for-of:

Fonction asynchrone logContent (urls) {
    pour (const url d'urls) {
        const content = attend httpGet (url);
        console.log (contenu);
    }
}
Maintenant tout est fini après la boucle for-of. Cependant, les étapes de traitement se déroulent de manière séquentielle: httpGet () n'est appelé qu'une deuxième fois après la fin du premier appel. Si vous souhaitez que les étapes de traitement se déroulent en parallèle, vous devez utiliser Promise.all ():

Fonction asynchrone logContent (urls) {
    attendez Promise.all (urls.map (
        async url => {
            const content = attend httpGet (url);
            console.log (contenu);
        }));
}
map () est utilisé pour créer un tableau de promesses. Nous ne sommes pas intéressés par les résultats qu'ils accomplissent, nous attendons seulement jusqu'à ce qu'ils soient tous remplis. Cela signifie que nous avons terminé complètement à la fin de cette fonction asynchrone. Nous pourrions tout aussi bien renvoyer Promise.all (), mai

s le résultat de la fonction serait un tableau dont les éléments sont tous indéfinis.


