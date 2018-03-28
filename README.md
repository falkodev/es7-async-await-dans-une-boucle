# ES7: async/await dans une boucle

`await` ne peut être utilisé qu'à l'intérieur d'une fonction `async`. C'est un problème lorsqu'on a besoin d'une des fonctions du prototype Array en JS telles que `map` ou `forEach` qui utilisent des callbacks.

## Array.prototype.map

Dans le code suivant, des urls dans un array sont téléchargées et renvoyées dans un autre array.

```js
async function downloadContent (urls) {
    return urls.map(url => {
        // Mauvaise syntaxe
        const content = await httpGet(url);
        return content;
    });
}
```

Cela ne fonctionne pas, car `await` a besoin d'être dans une fonction asynchrone `async`. On pourrait penser qu'il suffit d'ajouter `async` dans la boucle :

```js
async function downloadContent (urls) {
    return urls.map(async (url) => {
        const content = attend httpGet(url);
        return content;
    });
}
```

Il y a deux problèmes avec ce code:

Le résultat est maintenant un tableau de promesses, pas un tableau de string.
Les callbacks ne sont pas achevées une fois que `map()` est terminé, car `await` n'interrompt que sa fonction parente et que `httpGet()` est résolu de manière asynchrone. 

Nous pouvons résoudre les deux problèmes via `Promise.all`, qui convertit un tableau de promesses en une sorte de promesse parente :

```js
async function downloadContent (urls) {
    const promiseArray = urls.map(url => httpGet (url));
    return Promise.all(promiseArray);
}
```

## Array.prototype.forEach

Avec `forEach()`, voici comment traiter le même exemple :

```js
async function logContent (urls) {
    urls.forEach(url => {
        // Mauvaise syntaxe
        const content = await httpGet (url);
        console.log(content);
    });
}
```

Encore une fois, ce code produira une erreur de syntaxe, car on ne peut pas utiliser `await` dans les fonctions synchrones.

Avec une fonction asynchrone:

```js
async function logContent (urls) {
    urls.forEach(async url => {
        const content = await httpGet(url);
        console.log(content);
    });
    // Promesses pas encore résolues ici
}
```

Cela fonctionne techniquement, mais il faut savoir une chose : la promesse retournée par `httpGet` est résolue de manière asynchrone, ce qui signifie que les callbacks ne sont pas terminés lorsque `forEach` se termine. Par conséquent, `logContent` ne pourras pas être `await` par une autre fonction si besoin.

Dans ce cas, une boucle `for-of` est nécessaire :

```js
async function logContent (urls) {
    for (const url of urls) {
        const content = await httpGet(url);
        console.log(content);
    }
}
```

Désormais, les promesses de `logContent` sont résolues à la sortie de la boucle `for-of`. Cependant, les étapes de traitement se déroulent de manière séquentielle : le 2e appel à `httpGet` n'est appelé qu'après la fin du premier appel du 1er passage dans la boucle. 

En cas de besoin de parallélisation, il faut utiliser `Promise.all` : 

```js
async function logContent (urls) {
    await Promise.all(urls.map(
        async url => {
            const content = await httpGet(url);
            console.log(content);
        }));
}
```

`map` est utilisé ici pour créer un tableau de promesses qui se remplit au fur et à mesure. Les promesses seront résolues à la fin de la fonction.

## Conclusion

Pour résumer, la méthode `forEach` lance les promesses sans attendre leur résolution, elle est donc fortement déconseillée. Il vaut mieux utiliser une boucle `for-of` pour un déroulé séquentiel de plusieurs promesses. Et `Promise.all` avec `map` pour des résolutions en parallèle.


