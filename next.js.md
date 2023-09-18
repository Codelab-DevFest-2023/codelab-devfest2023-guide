id:docs

# Rendu front, action ! Découvrez les différents modes de rendu avec Next.js

## Initialisation du codelab

Duration: 5:00

Prérequis :

- [Git](https://git-scm.com/)
- [Node / npm](https://nodejs.org/) (version 18 minimum)
- [Visual Studio Code](https://code.visualstudio.com/) (ou tout autre IDE JS)

<aside class="negative">Il est impératif d'avoir une version 18 minimale de Node.js car elle est un prérequis pour les React Server Components que nous verrons dans ce codelab.</aside>

Pour ce codelab nous utiliserons deux repositories :

- L'un qui servira pour l'exercice sur le client side rendering
- L'autre servira pour les exercices suivants basés sur Next.js

Installer le projet pour l'exercice sur le client side rendering, en se positionnant sur la branche `start` :

```bash
git clone --branch start https://github.com/Codelab-DevFest-2023/codelab-devfest2023-csr.git
cd codelab-devfest2023-csr
npm install
```

Lancer la commande `npm run dev` et vérifier qu'on a bien une application qui s'affiche sur l'url `http://localhost:3000`.

Puis installer le projet pour les exercices avec Next.js :

```bash
git clone --branch start https://github.com/Codelab-DevFest-2023/codelab-devfest2023-next.git
cd codelab-devfest2023-next
npm install
```

Lancer la commande `npm run dev` et vérifier qu'on a bien une application qui s'affiche sur l'url `http://localhost:3000`.

## Client Side Rendering

Duration: 20:00

Dans cette première partie, nous allons développer une application en Client Side Rendering s'appuyant sur Vite.js, React et Tailwind. Nous n'utiliserons donc pas Next.js dans cet exercice, mais il permettra de se rendre compte des différences entre le client side rendering et les autres modes de rendu que nous verrons plus tard.

<aside>Pour se concentrer sur ce qui est spécifique au client side rendering, certains composants vous sont déjà fournis dans le repository : carte d'un film, méthodes de fetch, composant de layout, etc...</aside>

### Liste des films

Pour commencer, nous allons créer un écran affichant la liste des films. Nous avons donc besoin de créer :

- Un hook `useMovies` qui permet de charger la liste des films
- Un composant `MoviesList` qui affiche la liste des films
- Une route `/movies` associée au composant `MoviesList`

Dans le répertoire `/src/hooks` du projet créons le fichier `useMovies.ts`. Ce hook s'appuie sur la librairie React Query pour gérer du cache.

```ts
import { useQuery } from "@tanstack/react-query";

export default useMovies = () => {
  return useQuery({});
};
```

Puis dans le répertoire `/src/pages` nous allons créer le fichier `MoviesListPage.tsx` qui utilise le hook `useMovies` pour récupérer les films puis parcourt les films pour afficher une card pour chaque film. Un loader est affiché pendant le chargement des films.

```ts
TODO;
```

Enfin nous allons créer une route `/movies` dans le fichier `/src/router.ts`. Cette route est associée au composant `MoviesListPage`.

```ts
TODO;
```

Nous pouvons démarrer l'application avec la commande `npm run dev` et constater que la liste des films s'affiche correctement.

### Détail d'un film

Ensuite nous allons créer un écran affichant le détail de chaque film. Nous avons besoin de créer :

- Un hook `useMovie` qui permet de charger le détail du film
- Un composant `MovieDetailsPage` qui définit la page de détail d'un film
- Une route `/movies/:id` qui définit des routes pour le

Dans le répertoire `/src/hooks` du projet créons le fichier `useMovie.ts` avec le contenu suivant :

```ts
import { useQuery } from "@tanstack/react-query";

export default useMovie = () => {
  return useQuery({...});
};
```

Puis dans le répertoire `/src/pages` nous allons créer le fichier `MovieDetailsPage.tsx` qui utilise le hook `useMovie` pour récupérer le détail du film puis afficher les informations. Un loader est affiché pendant le chargement du film.

```ts
TODO;
```

Enfin nous allons créer une route `/movies/:id` dans le fichier `/src/router.ts`. Cette route est associée au composant `MovieDetailsPage`.

```ts
TODO;
```

Nous pouvons démarrer l'application avec la commande `npm run dev` et constater que la liste des films s'affiche correctement.

### Analysons cette application

Ouvrons les devtools de Chrome (ou équivalent) pour observer :

- Les chargements de fichiers Javascript
- Les appels de service REST

Enfin affichons le code source de la page pour constater que cette page n'est pas très "SEO friendly" !

## Server Side Rendering

Duration: 20:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "step3" pour débuter cette étape : <code>git checkout -f step3</code></aside>

### Liste des films

TODO

### Détail d'un film

TODO

### Recherche de films

TODO

### Gestion de favoris

TODO

### Observons ce qui se passe

TODO

## Static Site Generation

Duration: 25:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "step4" pour débuter cette étape : <code>git checkout -f step4</code></aside>

### Liste des films

TODO

### Détail d'un film

TODO

### Optimisation des images

TODO

## React Server Components

Duration: 15:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "step5" pour débuter cette étape : <code>git checkout -f step5</code></aside>

### Liste des films

TODO

### Détail d'un film

TODO

### Recherche de films

TODO

### Gestion de favoris

TODO

### Affichage de la note

TODO La note
