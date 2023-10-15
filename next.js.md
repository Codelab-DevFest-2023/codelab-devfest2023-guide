id:docs

# Rendu front, action ! Découvrez les différents modes de rendu avec Next.js

## Initialisation du codelab

Duration: 10:00

Prérequis :

- [Git](https://git-scm.com/)
- [Node / npm](https://nodejs.org/) (version 18 minimum)
- [Visual Studio Code](https://code.visualstudio.com/) (ou tout autre IDE JS)

<aside class="negative">Il est impératif d'avoir une version 18 minimale de Node.js car elle est un prérequis pour les React Server Components que nous verrons dans ce codelab.</aside>

Pour ce codelab nous utiliserons deux repositories :

- Le premier servira pour l'exercice sur le client side rendering
- L'autre servira pour les exercices suivants, basés sur Next.js

Clonons le projet pour l'exercice sur le client side rendering, en se positionnant sur la branche `start` :

```bash
git clone --branch start https://github.com/Codelab-DevFest-2023/codelab-devfest2023-csr.git
cd codelab-devfest2023-csr
npm install
```

Lançons la commande `npm run dev` et vérifions qu'on a bien une application qui s'affiche sur l'url `http://localhost:5173`.

Puis installons le projet pour les exercices avec Next.js :

```bash
git clone --branch start https://github.com/Codelab-DevFest-2023/codelab-devfest2023-next.git
cd codelab-devfest2023-next
npm install
```

Lançons la commande `npm run dev` et vérifons qu'on a bien une application qui s'affiche sur l'url `http://localhost:3000`.

## Client Side Rendering

Duration: 25:00

Dans cette première partie, nous allons développer une application en Client Side Rendering (CSR) s'appuyant sur Vite.js, React et Tailwind. Nous n'utiliserons donc pas Next.js dans cet exercice. Mais en créant une application en CSR, nous verrons mieux les différences avec les autres modes de rendu que nous verrons plus tard.

Pour cette partie nous travaillerons dans le répertoire `codelab-devfest2023-csr`.

<aside>Pour se concentrer sur ce qui est spécifique au client side rendering, certains éléments sont déjà fournis dans le repository : carte d'un film, méthodes de fetch des données, composant de layout, etc...</aside>

### Création des variables d'environnement

Pour pouvoir utiliser l'API TMDB nous devons fournir une clé d'API. Créons un fichier `.env` à la racine du projet avec le contenu suivant :

```
VITE_API_URL=https://api.themoviedb.org/3
VITE_API_KEY=xxx
```

Utilisons une clé d'API parmi celles-ci :

Clé 1 :

```
eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIyZmMxMDQ0YjI0ZDdkYjcyY2RlZmJmNTBkNTkyNzhiYyIsInN1YiI6IjY0YzU2ZTgyNjNhNjk1MDEwMzk5Y2I0YiIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.Rc42SZEbnqH6PvJRj1GAVYTADLcR5vNzArE1P333_dI
```

Clé 2 :

```
eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIxNTk0NTM5OGIxMzZhODZhZDU4ZjE3MmFlYTQ3ZTNjZCIsInN1YiI6IjVlNzkxY2Y0MzU3YzAwMDAxMTU0MDAwZSIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.8XGXQO22Mv8UaooWf0sDnUWRpNTdlzuO5rZClx-eEDc
```

Clé 3 :

```
eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJmNTA3Yzc1MTA3MWQyZjkzZTRiMTU2Mzg4NzY3NjEwMSIsInN1YiI6IjY1MjNmYzBjNzQ1MDdkMDBlMjEzYjMxNiIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.7Ge2En8f5fq1Iq1MyzIKPUL1ITqaDECKYhsi8UqSucQ
```

Clé 4 :

```
eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIyYjIxMDJiZjVhNDJlNmI4YmE5MTc0ZDFmY2ZjMzE1NCIsInN1YiI6IjY1MjNmZmRhMGNiMzM1MTZmNWM1ZWJmYSIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.e_DyQRJhs1aK8FPXbM-dgn025D-Xp2M9fXQ1GdEYGjw
```

### Liste des 20 films les plus populaires

Pour commencer, nous allons créer une page affichant la liste des 20 films les plus populaires du moment. Nous avons donc besoin de créer :

- Un hook `useMovies` qui permet de charger la liste des films
- Un composant `MoviesListPage` qui correspond à la page affichant la liste des films
- Une route `/movies` associée au composant `MoviesListPage`

Créons le fichier `/src/hooks/movies.ts` et ajoutons y le hook `useMovies`. Ce hook s'appuie sur la librairie React Query pour charger la liste des films et gérer du cache. Elle s'appuie sur un service de fetch qui appelle l'API TMDB.

```ts
import { useQuery } from '@tanstack/react-query';
import { AxiosError } from 'axios';
import { Movie } from '../interfaces/movie.interface';
import { fetchPopularMovies } from '../services/movie.service';

const useMovies = () => {
  return useQuery<Movie[], AxiosError>({
    queryKey: ['movies'],
    queryFn: () => fetchPopularMovies(),
  });
};

export { useMovies };
```

Puis modifions le composant `MoviesListPage` dans le fichier `/src/pages/movies/MoviesListPage.tsx` :

- Appel du hook `useMovies` pour récupérer les films
- Affichage de `Chargement ...` pendant le chargement des données
- Affichage d'un message d'erreur et d'un bouton `Réessayer`
- Affichage d'une carte pour chaque film chargé
- Affichage d'un message dans le cas où on n'a aucun film

Le composant doit ressembler à ça :

```ts
import MovieCard from '../../components/movie/card/MovieCard';
import { useMovies } from '../../hooks/movies';

const MoviesListPage = () => {
  const { data: movies, isFetching, isError, isFetched, refetch } = useMovies();
  return (
    <div className="main-container">
      {isError && (
        <div>
          <p className="text-red mb-4">
            Erreur lors de la récupération des films ...
          </p>
          <button
            type="button"
            onClick={() => {
              refetch();
            }}
          >
            Réessayer
          </button>
        </div>
      )}
      {isFetching && <p>Chargement...</p>}
      {isFetched && movies && movies?.length > 0 && (
        <ul className="movies-list">
          {movies.map((movie) => (
            <li key={movie.id}>
              <MovieCard movie={movie} />
            </li>
          ))}
        </ul>
      )}
      {isFetched && movies && movies.length < 1 && (
        <p className="font-medium text-3xl">Aucun résultat</p>
      )}
    </div>
  );
};

export default MoviesListPage;
```

Enfin nous allons ajouter une route `/movies` dans le fichier `/src/App.tsx` et l'associer au composant `MoviesListPage` comme ci-dessous.

```ts
import { BrowserRouter, Route, Routes } from 'react-router-dom';
import Layout from './components/layout/Layout';
import HomePage from './pages/home/HomePage';
import MoviesListPage from './pages/movies/MoviesListPage';

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<HomePage />} />
          <Route path="movies" element={<MoviesListPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
};

export default App;
```

Il ne reste plus qu'à rajouter un accès vers notre nouvelle route dans le fichier `/src/components/header/Header.tsx`

```ts
// ...
          <span className="header-title">
            Rendu front, action !
          </span>
        </a>

        <ul className="header-item">
          <li>
            <Link
              to="/movies"
              className="header-link"
            >
              Liste des films
            </Link>
          </li>
        </ul>

        <img
          className="lg:block hidden"
// ...
```

Démarrons l'application en mode développement avec la commande `npm run dev`, ouvrons `http://localhost:5173`, cliquons sur le lien "Liste des films" dans le header et vérifions que la liste des films s'affiche correctement.

### Détail d'un film

Ensuite nous allons créer une page affichant le détail de chaque film. Nous avons besoin de créer :

- Un hook `useMovie` qui permet de charger le détail du film
- Un composant `MovieDetailsPage` qui définit la page de détail d'un film
- Une route `/movies/:id` qui définit la route permettant d'afficher le détail de chaque film

Modifions le fichier `/src/hooks/movies.ts` pour ajouter un nouveau hook `useMovie` qui charge le détail d'un film (toujours en s'appuyant sur React Query) :

```ts
import { useQuery } from '@tanstack/react-query';
import { AxiosError } from 'axios';
import { Movie } from '../interfaces/movie.interface';
import {
  fetchMovieDetails,
  fetchPopularMovies,
} from '../services/movie.service';

// ...

const useMovie = (movieId: number) => {
  return useQuery<Movie, AxiosError>({
    queryKey: ['movies', movieId],
    queryFn: () => fetchMovieDetails(movieId),
  });
};

export { useMovie, useMovies };
```

Puis modifions le fichier `/src/pages/movies/MovieDetailsPage.tsx` :

- Utilisation du hook `useMovie` pour récupérer le détail du film
- Utilisation des statuts et méthodes retournés par React Query pour gérer un message de chargement et gérer les erreurs de chargement
- Affichage des informations sur le film

```ts
import { useParams } from 'react-router-dom';
import { useMovie } from '../../hooks/movies';
import Note from '../../components/note/Note';

const MovieDetailsPage = () => {
  const { movieId } = useParams();
  const {
    data: movie,
    isFetching,
    isError,
    isFetched,
    refetch,
  } = useMovie(Number(movieId));

  const posterUrl = `https://image.tmdb.org/t/p/w500/${movie?.poster_path}`;
  const backdropPathUrl = `https://image.tmdb.org/t/p/original/${movie?.backdrop_path}`;

  return (
    <div className="details-page">
      {isError && (
        <div>
          <p className="text-red mb-4">
            Erreur lors de la récupération du film ...
          </p>
          <button type="button" onClick={() => refetch()}>
            Réessayer
          </button>
        </div>
      )}
      {isFetching && <p>Chargement...</p>}
      {isFetched && movie && (
        <>
          <div className="poster">
            <img
              src={posterUrl}
              alt={movie.title}
              className="poster-image"
              height={750}
              width={500}
              sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            />
          </div>
          <div className="relative-full">
            <img
              className="movie-backdrop"
              alt={movie.title}
              src={backdropPathUrl}
            />
            <div className="movie-description">
              <div className="informations">
                <h1 className="movie-title">{movie.title}</h1>
                <div className="movie-genre">
                  {movie.genres.map((genre) => {
                    return <p key={genre.id}>{genre.name}</p>;
                  })}
                </div>
                <p>{movie.tagline}</p>
                <p className="movie-overview">{movie.overview}</p>
                <div className="movie-note">
                  <Note note={movie.vote_average} />
                </div>
              </div>
            </div>
          </div>
        </>
      )}
    </div>
  );
};

export default MovieDetailsPage;
```

Créons une nouvelle route `/movies/:id` dans le fichier `/src/App.tsx`.

```ts
import { BrowserRouter, Route, Routes } from 'react-router-dom';
import Layout from './components/layout/Layout';
import HomePage from './pages/home/HomePage';
import MovieDetailsPage from './pages/movies/MovieDetailsPage';
import MoviesListPage from './pages/movies/MoviesListPage';

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<HomePage />} />
          <Route path="movies" element={<MoviesListPage />} />
          <Route path="movies/:movieId" element={<MovieDetailsPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
};

export default App;
```

Et modifions la liste des films (`src/pages/movies/MoviesListPage.tsx`) pour ajouter un lien vers la page de détail du film sur la carte `MovieCard`, en utilisant le composant `Link` de `react-router-dom`.

```ts
{
  movies.map((movie) => (
    <li key={movie.id}>
      <Link to={`/movies/${movie.id}`}>
        <MovieCard movie={movie} />
      </Link>
    </li>
  ));
}
```

Nous pouvons démarrer l'application avec la commande `npm run dev`, ouvrir le browser sur `http://localhost:5173`, accéder à liste des films puis cliquer sur un film afin d'accéder à la page de détail de chaque film.

### Analysons cette application

Pour analyser le comportement d'une application client side rendering, nous allons créer une version de production de l'application avec la commande `npm run build` puis la lancer en mode preview avec la commande `npm run preview`.

Ouvrons l'application sur `http://localhost:4173` puis ouvrons les devtools de Chrome (ou équivalent) pour observer :

- Les chargements de fichiers Javascript
- Les appels de service REST

Enfin affichons le code source de la page pour constater que cette page n'est pas très "SEO friendly" !

## Server Side Rendering

Duration: 25:00

Dans cette deuxième partie, nous allons développer une application en Server Side Rendering s'appuyant sur Next.js et Tailwind.

Pour cette partie et les suivantes nous travaillerons dans le répertoire `codelab-devfest2023-next`.

<aside>Pour se concentrer sur ce qui est spécifique au server side rendering, certains composants vous sont déjà fournis dans le repository : carte d'un film, méthodes de fetch, composant de layout, etc...</aside>

<aside class="negative">Cet exercice utilisera le système de routage <strong>pages</strong> de Next.js.</aside>

### Création des variables d'environnement

Comme dans l'exercice précédent nous avons besoin de fournir des variables d'environnement pour fournir la clé API nécessaire à l'utilisation de l'API TMDB. Créons un fichier `.env.local` à la racine du projet avec le contenu suivant et remplaçons par la même clé API que celle utilisée lors de l'exercice précédent :

```
NEXT_PUBLIC_API_URL=https://api.themoviedb.org/3
NEXT_PUBLIC_API_KEY=eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIxNTk0NTM5OGIxMzZhODZhZDU4ZjE3MmFlYTQ3ZTNjZCIsInN1YiI6IjVlNzkxY2Y0MzU3YzAwMDAxMTU0MDAwZSIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.8XGXQO22Mv8UaooWf0sDnUWRpNTdlzuO5rZClx-eEDc
```

### Liste des films

Nous allons créer une page affichant la liste des films.

Dans le répertoire `/src/pages/ssr` du projet, modifions le fichier `index.tsx`.

Nous allons définir une méthode `getServerSideProps` qui s'exécute côté serveur lorsque la page est générée. Ici elle permet d'appeler l'API pour récupérer la liste des films et la passer en tant que props `movies` au composant `SSRPage`.

```tsx
import { Movie } from '@/interfaces/movie.interface';
import { fetchPopularMovies } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';

const SSRPage = ({
  movies,
}: InferGetServerSidePropsType<typeof getServerSideProps>) => {
  // ...
};

export const getServerSideProps: GetServerSideProps<{
  movies: Movie[];
}> = async () => {
  const { results: movies } = await fetchPopularMovies();
  return { props: { movies } };
};

export default SSRPage;
```

Il ne reste plus qu'à finir de construire notre page, en parcourant la liste de films fournie par la props `movies` :

```tsx
import MovieCard from '@/components/movie/card/MovieCard';
import { Movie } from '@/interfaces/movie.interface';
import { fetchPopularMovies } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';

const SSRPage = ({
  movies,
}: InferGetServerSidePropsType<typeof getServerSideProps>) => {
  return (
    <>
      <Head>
        <title>Server Side Rendering</title>
      </Head>
      <main className="main-container">
        <ul className="movies-list">
          {movies?.map((movie: Movie) => (
            <li key={movie.id}>
              <MovieCard movie={movie} />
            </li>
          ))}
        </ul>
        {movies.length < 1 && (
          <p className="font-medium text-3xl">Aucun résultat</p>
        )}
      </main>
    </>
  );
};

// ...
```

Nous pouvons démarrer l'application avec la commande `npm run dev` et constater que la liste des films s'affiche correctement à l'adresse `http://localhost:3000/ssr`.

### Optimisations des images

Afin d'optimiser l'affichage des images sur l'application, nous allons utiliser le composant `Image` de Next.js.

Ce composant intègre des optimisations sur la gestion des images pour améliorer l'expérience utilisateur et l'expérience développeur : chargement en mode lazy, chargement prioritaire, gestion du responsive, etc...

Dans le répertoire `/src/components/movie/card` du projet, ouvrons le composant `MovieCard`.

Remplaçons le composant `img` par un composant `Image` :

```tsx
import Image from 'next/image';

// ...

<Image
  src={posterUrl}
  alt={movie.title}
  className="movie-poster"
  height={750}
  width={500}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>;

// ...
```

### Recherche de films

Nous allons créer un composant qui affiche un champ de recherche de films. Le terme recherché sera ajouté en tant que paramètre `searchKey` dans l'url de la page : `http://localhost:3000/ssr?searchKey=xxxx`.

Un squelette du composant `SearchBox` est déjà fourni dans le fichier `/src/components/search/SearchBox.tsx`. Nous allons compléter ce composant pour :

- Gérer la valeur saisie dans le champ de recherche
- Utiliser les hooks de navigation de Next.js pour rafraîchir la page en passant dans le paramètre `searchKey` de l'url le terme recherché.

Tout le code ajouté est basé sur des hooks et des méthodes événementielles, il ne s'exécutera donc que dans le browser, côté client.

Voici le code du composant terminé :

```tsx
import { usePathname, useRouter, useSearchParams } from 'next/navigation';
import { ChangeEvent, useState } from 'react';

const SearchBox = () => {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();
  const [searchValue, setSearchValue] = useState<string | undefined>();

  const handleSearchChange = (event: ChangeEvent<HTMLInputElement>) => {
    const currentPathname = pathname ?? '/';
    const value = event.target.value;
    setSearchValue(value);

    if (value.length > 3) {
      const newUrl = currentPathname + '?searchKey=' + encodeURI(value);
      router.push(newUrl);
    }

    if (value.length <= 2 && searchParams?.get('searchKey')) {
      router.push(currentPathname);
    }
  };

  return (
    <input
      type="text"
      className="search-input"
      placeholder="Recherche ..."
      value={searchValue}
      onChange={handleSearchChange}
    />
  );
};

export default SearchBox;
```

Ce paramètre `searchKey` doit être récupéré côté serveur pour appeler la bonne route d'API TMDB :

- La route `search/movie` en cas de présence du paramètre `searchKey`
- La route `movie/popular` en cas d'absence du paramètre `searchKey`

Modifions donc la méthode `getServerSideProps` de la page `/src/pages/ssr/index.tsx` afin de récupérer le paramètre `searchKey` et le passer à la méthode `searchMovies` :

```tsx
import MovieCard from '@/components/movie/card/MovieCard';
import SearchBox from '@/components/search/SearchBox';
import { Movie } from '@/interfaces/movie.interface';
import { fetchPopularMovies, searchMovies } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';
import Link from 'next/link';

// ...

export const getServerSideProps: GetServerSideProps<{
  movies: Movie[];
}> = async ({ query }) => {
  let movies: Movie[];

  if (query.searchKey) {
    const { results } = await searchMovies(query.searchKey as string);
    movies = results;
  } else {
    const { results } = await fetchPopularMovies();
    movies = results;
  }

  return { props: { movies } };
};
```

Il ne reste plus qu'a ajouter le composant `SearchBox` dans notre page `src/pages/ssr/index.tsx` :

```tsx
import MovieCard from '@/components/movie/card/MovieCard';
import SearchBox from '@/components/search/SearchBox';
import { Movie } from '@/interfaces/movie.interface';
import { fetchPopularMovies, searchMovies } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';
import Link from 'next/link';

// ...
<>
  <Head>
    <title>Server Side Rendering</title>
  </Head>
  <main className="main-container">
    <SearchBox />
    <ul className="movies-list">
      {movies?.map((movie: Movie) => (
        <li key={movie.id}>
          <MovieCard movie={movie} />
        </li>
      ))}
    </ul>
    {movies.length < 1 && (
      <p className="font-medium text-3xl">Aucun résultat</p>
    )}
  </main>
</>;
// ...
```

Nous pouvons redémarrer l'application avec la commande `npm run dev` et constater que la recherche s'effectue correctement à l'adresse `http://localhost:3000/ssr`.

### Détail d'un film

Nous allons ensuite créer une page affichant le détail de chaque film.

Dans le répertoire `/src/pages/ssr` du projet, modifions le fichier `[id].tsx`

Modifions la méthode `getServerSideProps` pour récupérer les données concernant le film passé en paramètre.

```tsx
import { Movie } from '@/interfaces/movie.interface';
import { fetchMovieDetails } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';

const SSRMovieDetailsPage = ({
  movie,
}: InferGetServerSidePropsType<typeof getServerSideProps>) => {
  // ...
};

export const getServerSideProps: GetServerSideProps<{
  movie: Movie;
}> = async ({ params }) => {
  if (params?.id) {
    const id = Number(params.id);
    const movie = await fetchMovieDetails(id);
    return { props: { movie } };
  } else {
    throw new Error('Missing id parameter');
  }
};

export default SSRMovieDetailsPage;
```

Il ne reste plus qu'à finir de construire notre page, en affichant :

- L'affiche du film
- Le titre du film
- Le synopsis du film
- La note du film

```tsx
import Note from '@/components/note/Note';
import { Movie } from '@/interfaces/movie.interface';
import { fetchMovieDetails } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';
import Image from 'next/image';

const SSRMovieDetailsPage = ({
  movie,
}: InferGetServerSidePropsType<typeof getServerSideProps>) => {
  const posterUrl = `https://image.tmdb.org/t/p/w500/${movie.poster_path}`;
  const backdropPathUrl = `https://image.tmdb.org/t/p/original/${movie.backdrop_path}`;

  return (
    <main>
      <Head>
        <title>Server Side Rendering</title>
      </Head>
      <div className="details-page">
        <div className="poster">
          <Image
            src={posterUrl}
            alt={movie.title}
            className="poster-image"
            height={750}
            width={500}
            sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            priority
          />
        </div>
        <div className="relative-full">
          <Image
            className="movie-backdrop"
            alt={movie.title}
            src={backdropPathUrl}
            fill
            priority
          />
          <div className="movie-description">
            <div className="informations">
              <h1 className="movie-title">{movie.title}</h1>
              <div className="movie-genre">
                {movie.genres.map((genre) => {
                  return <p key={genre.id}>{genre.name}</p>;
                })}
              </div>
              <p>{movie.tagline}</p>
              <p className="movie-overview">{movie.overview}</p>
              <div className="movie-note">
                <Note note={movie.vote_average} />
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  );
};

// ...
```

Et enfin modifions la liste des films (`src/pages/ssr/index.tsx`) pour ajouter un lien vers la page de détail du film sur la carte `MovieCard`, en utilisant le composant `Link` de `next/link`.

```ts
{
  movies?.map((movie: Movie) => (
    <li key={movie.id}>
      <Link href={`/ssr/${movie.id}`}>
        <MovieCard movie={movie} />
      </Link>
    </li>
  ));
}
```

Vérifions dans l'application qu'on accède bien au détail d'un film quand on clique sur sa vignette dans la liste des films.

### Observons ce qui se passe

Construisons une version de production de l'application avec la commande `npm run build` puis démarrons l'application avec la commande `npm run start`.

Ouvrons l'application sur `http://localhost:3000` puis accédons à la rubrique "Server Side Rendering" et observons ce qui se passe dans les devtools de Chrome (ou équivalent) :

- Les chargements de fichiers Javascript
- Les appels de service REST

## Static Site Generation

Duration: 15:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "ssr-end" pour débuter cette étape : <code>git checkout -f ssr-end</code></aside>

Dans cette troisième partie, nous allons développer la même application que précédement mais en Static Site Generation en s'appuyant sur Next.js et Tailwind.

<aside class="negative">Cet exercice utilisera le routage <strong>pages</strong> (Next.js < 13).</aside>

### Liste des films

Comme dans l'exercice précédent, nous aurons besoin de créer la page d'affichage de la liste des films.

Dans le répertoire `/src/pages/ssg` du projet, modifions le fichier `index.tsx`.

Il sera très semblable au fichier `/src/pages/ssr`. La principale différence réside dans le remplacement de méthode `getServerSideProps` par la méthode `getStaticProps`.

```ts
export const getStaticProps: GetStaticProps<{
  movies: Movie[];
}> = async () => {
  const { results: movies } = await fetchPopularMovies();
  return { props: { movies } };
};
```

Le reste du fichier est très semblable à ce qui a été fait pour le server side rendering : on passe la liste des films dans la props `movies` du composant et on parcourt cette liste.

```tsx
import MovieCard from '@/components/movie/card/MovieCard';
import { Movie } from '@/interfaces/movie.interface';
import { fetchPopularMovies } from '@/services/movie.service';
import {
  GetServerSideProps,
  GetStaticProps,
  InferGetServerSidePropsType,
} from 'next';
import Head from 'next/head';
import Link from 'next/link';

const SSGPage = ({
  movies,
}: InferGetServerSidePropsType<typeof getStaticProps>) => {
  return (
    <>
      <Head>
        <title>Static Site Generation</title>
      </Head>
      <main className="main-container">
        <ul className="movies-list">
          {movies?.map((movie: Movie) => (
            <li key={movie.id}>
              <Link href={`/ssg/${movie.id}`}>
                <MovieCard movie={movie} />
              </Link>
            </li>
          ))}
        </ul>
        {movies.length < 1 && (
          <p className="font-medium text-3xl">Aucun résultat</p>
        )}
      </main>
    </>
  );
};

export const getStaticProps: GetStaticProps<{
  movies: Movie[];
}> = async () => {
  const { results: movies } = await fetchPopularMovies();
  return { props: { movies } };
};

export default SSGPage;
```

Nous pouvons démarrer l'application avec la commande `npm run dev` et constater que la liste des films s'affiche correctement à l'adresse `http://localhost:3000/ssg`.

### Détail d'un film

Ensuite, nous allons créer les pages affichant le détail de chaque film. Là encore le code est très proche de ce qu'on a en Server Side Rendering.

Modifions le fichier `/src/pages/ssg/[id].tsx` du projet et ajoutons la méthode `getStaticProps`.

```ts
export const getStaticProps: GetStaticProps<{
  movie: Movie;
}> = async ({ params = {} }) => {
  const id = Number(params.id);
  const movie = await fetchMovieDetails(id);
  return { props: { movie } };
};
```

Le reste du fichier est quasiment identique au server side rendering :

```ts
import Note from '@/components/note/Note';
import { Movie } from '@/interfaces/movie.interface';
import {
  fetchMovieDetails,
  fetchPopularMovies,
} from '@/services/movie.service';
import { GetStaticProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';
import Image from 'next/image';

const SSGMovieDetailsPage = ({
  movie,
}: InferGetServerSidePropsType<typeof getStaticProps>) => {
  const posterUrl = `https://image.tmdb.org/t/p/w500/${movie.poster_path}`;
  const backdropPathUrl = `https://image.tmdb.org/t/p/original/${movie.backdrop_path}`;

  return (
    <main>
      <Head>
        <title>Static Site Generation</title>
      </Head>
      <div className="details-page">
        <div className="poster">
          <Image
            src={posterUrl}
            alt={movie.title}
            className="poster-image"
            height={750}
            width={500}
            sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            priority
          />
        </div>
        <div className="relative-full">
          <Image
            className="movie-backdrop"
            alt={movie.title}
            src={backdropPathUrl}
            fill
            priority
          />
          <div className="movie-description">
            <div className="informations">
              <h1 className="movie-title">{movie.title}</h1>
              <div className="movie-genre">
                {movie.genres.map((genre) => {
                  return <p key={genre.id}>{genre.name}</p>;
                })}
              </div>
              <p>{movie.tagline}</p>
              <p className="movie-overview">{movie.overview}</p>
              <div className="movie-note">
                <Note note={movie.vote_average} />
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  );
};

export const getStaticProps: GetStaticProps<{
  movie: Movie;
}> = async ({ params = {} }) => {
  const id = Number(params.id);
  const movie = await fetchMovieDetails(id);
  return { props: { movie } };
};

export default SSGMovieDetailsPage;
```

Nous avons cependant besoin d'ajouter une méthode supplémentaire dans ce fichier : la méthode `getStaticPaths` retourne la liste des URLs des pages de détail à créer en s'appuyant sur la liste des films.

```ts
import {
  GetStaticPaths,
  GetStaticProps,
  InferGetServerSidePropsType,
} from 'next';

// ...

export const getStaticPaths: GetStaticPaths = async () => {
  const { results: movies } = await fetchPopularMovies();
  return {
    paths: movies.map((movie) => `/ssg/${movie.id}`),
    fallback: false,
  };
};
```

<aside>On voit ici que vu qu'on génère une page pour chaque film, le build de l'application en mode Static Site Generation peut être long si on s'intéresse à l'ensemble des films de la base TMDB ! C'est pour cette raison que dans cet exercice on s'est intéressés uniquement aux films populaires du moment.</aside>

### Observons ce qui se passe ...

Pour réellement comprendre le fonctionnement du Static Site Generation, il faut lancer le build du site avec la commande `npm run build`.
Observons alors que des requêtes sont envoyées à l'API TMDB pour construire des pages statiques.

Exécutons ensuite la commande `npm run start` pour lancer l'application. Le résultat pour l'utilisateur est rigoureusement identique.

## React Server Components

Duration: 25:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "ssg-end" pour débuter cette étape : <code>git checkout -f ssg-end</code></aside>

Dans cette quatrième et dernière partie, nous allons développer la même application que précédement mais en utilisant les React Server Components, toujours en s'appuyant sur Next.js et Tailwind.

<aside class="negative">Cet exercice utilisera le routage <strong>app</strong> (Next.js > 13).</aside>

### Liste des films

Comme dans l'exercice précédent, nous aurons besoin de créer la page d'affichage de la liste des films.

Dans le répertoire `/src/app/rsc` du projet, modifions le fichier `page.tsx`.

Nous n'avons plus besoin d'utiliser une méthode spécifique pour récupérer les données. nous pouvons directement faire un appel API dans notre composant serveur :

```tsx
import MovieCard from '@/components/movie/card/MovieCard';
import SearchBox from '@/components/search/SearchBox';
import { Movie } from '@/interfaces/movie.interface';
import { fetchPopularMovies, searchMovies } from '@/services/movie.service';
import Link from 'next/link';

export const revalidate = 0;

interface Props {
  searchParams: { [key: string]: string | string[] | undefined };
}

const RSCPage = async ({ searchParams }: Props) => {
  let movies: Movie[];

  if (searchParams.searchKey) {
    const { results } = await searchMovies(searchParams.searchKey as string);
    movies = results;
  } else {
    const { results } = await fetchPopularMovies();
    movies = results;
  }

  return (
    <main className="main-container">
      <SearchBox />
      <ul className="movies-list">
        {movies?.map((movie: Movie) => (
          <li key={movie.id}>
            <Link href={`/rsc/${movie.id}`}>
              <MovieCard movie={movie} />
            </Link>
          </li>
        ))}
      </ul>
      {movies.length < 1 && (
        <p className="font-medium text-3xl">Aucun résultat</p>
      )}
    </main>
  );
};

export default RSCPage;
```

<aside>Le code <strong>export const revalidate = 0;</strong> est un mécanisme qui permet de mettre en cache les données générées côté serveur (SSR) pendant une période spécifiée.
En mettant revalidate à zéro (0), cela signifie que les données seront générées à chaque demande sans mise en cache ni expiration.
</aside>

Nous pouvons redémarrer l'application avec la commande `npm run dev` et constater que la liste des films s'affiche correctement à l'adresse `http://localhost:3000/rsc`.

<aside class="negative">
Lors du lancement de l'application, vous devriez avoir une error Nextjs.
Avec la version 13, <strong>"use client"</strong> est toujours nécessaire lorsque le composant est rendu sur le client
</aside>

Il est impératif de spécifier 'use client' dans le fichier `/src/components/search/SearchBox.tsx`.

```tsx
'use client';
import { usePathname, useRouter, useSearchParams } from 'next/navigation';
import { ChangeEvent, useState } from 'react';

const SearchBox = () => {
// ...
```

### Détail d'un film

Ensuite, nous allons créer un écran affichant le détail de chaque film.

Dans le répertoire `/src/app/rsc/[id]` du projet, modifions le fichier `page.tsx`.

Comme précédement, nous pouvons directement faire un appel API dans notre composant serveur :

```tsx
import Note from '@/components/note/Note';
import { fetchMovieDetails } from '@/services/movie.service';
import Image from 'next/image';

const RSCMovieDetailsPage = async ({ params }: { params: { id: number } }) => {
  const movie = await fetchMovieDetails(params.id);

  const posterUrl = `https://image.tmdb.org/t/p/w500/${movie.poster_path}`;
  const backdropPathUrl = `https://image.tmdb.org/t/p/original/${movie.backdrop_path}`;

  return (
    <main className="details-page">
      <div className="poster">
        <Image
          src={posterUrl}
          alt={movie.title}
          className="poster-image"
          height={750}
          width={500}
          sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
          priority
        />
      </div>
      <div className="relative-full">
        <Image
          className="movie-backdrop"
          alt={movie.title}
          src={backdropPathUrl}
          fill
          priority
        />
        <div className="movie-description">
          <div className="informations">
            <h1 className="movie-title">{movie.title}</h1>
            <div className="movie-genre">
              {movie.genres.map((genre) => {
                return <p key={genre.id}>{genre.name}</p>;
              })}
            </div>
            <p>{movie.tagline}</p>
            <p className="movie-overview">{movie.overview}</p>
            <div className="movie-note">
              <Note note={movie.vote_average} />
            </div>
          </div>
        </div>
      </div>
    </main>
  );
};

export default RSCMovieDetailsPage;
```

### Gestion de favoris

Nous allons mettre en place la possibilité de gérer des films favoris à travers un composant `Like` présent dans la page de détail qui permet d'ajouter / enlever le film des favoris, stockés dans le local storage du navigateur. Le local storage n'étant disponible que côté client, il faudra s'assurer que notre code de gestion des favoris ne s'exécute que côté client.

Dans le répertoire `/src/components/like` du projet, ouvrons le fichier `Like.tsx`.

```tsx
'use client';
import { useEffect, useState } from 'react';
import HeartIcon from './HeartIcon';

interface Props {
  id: number;
}

interface Favorite {
  id: number;
}

const Like = ({ id }: Props) => {
  const [isFavorite, setIsFavorite] = useState<boolean>(false);

  useEffect(() => {
    const favorites = localStorage.getItem('favorites');
    if (favorites) {
      const isFound = JSON.parse(favorites).find(
        (favorite: Favorite) => favorite.id === id
      );
      if (isFound) {
        setIsFavorite(true);
      }
    } else {
      setIsFavorite(false);
    }
  }, [id]);

  const handleClick = () => {
    const storedFavorites = localStorage.getItem('favorites');
    if (storedFavorites) {
      const favoritesList: Favorite[] = JSON.parse(storedFavorites);
      const hasFavorite = favoritesList.some(
        (favorite: Favorite) => favorite.id === id
      );

      if (!hasFavorite) {
        favoritesList.push({ id });
        localStorage.setItem('favorites', JSON.stringify(favoritesList));
        setIsFavorite(true);
      } else {
        const updatedFavoritesList = favoritesList.filter(
          (favorite: Favorite) => favorite.id !== id
        );
        localStorage.setItem('favorites', JSON.stringify(updatedFavoritesList));
        setIsFavorite(false);
      }
    } else {
      localStorage.setItem('favorites', JSON.stringify([{ id: id }]));
      setIsFavorite(true);
    }
  };

  return (
    <button
      className={`heart-button ${isFavorite ? 'filled' : ''}`}
      onClick={handleClick}
    >
      <HeartIcon />
    </button>
  );
};

export default Like;
```

Il ne reste plus qu'à rajouter ce nouveau composant dans la page RSC (`/src/app/rsc/[id]/page.tsx`) :

```tsx
import Like from '@/components/like/Like';
import Note from '@/components/note/Note';
import { fetchMovieDetails } from '@/services/movie.service';
import Image from 'next/image';

// ...
<div className="movie-description">
  <div className="informations">
    <h1 className="movie-title">{movie.title}</h1>
    <div className="movie-genre">
      {movie.genres.map((genre) => {
        return <p key={genre.id}>{genre.name}</p>;
      })}
    </div>
    <p>{movie.tagline}</p>
    <p className="movie-overview">{movie.overview}</p>
    <div className="movie-note">
      <Note note={movie.vote_average} />
      <Like id={movie.id} />
    </div>
  </div>
</div>;
// ...
```

Nous pouvons redémarrer l'application avec la commande `npm run dev` et vérifier le bon comportement de notre nouveau composant.

### Affichage des critiques de films

Afin de tirer partie de la force des React Server Components, nous allons afficher les critiques du film sur la page de détail.
Nous allons :

- Mettre en cache la requête de récupération du film (7 jours)
- Ne pas mettre de cache sur la requête de récupération des critiques
- Garder nos composants client

Dans le répertoire `/src/services` du projet, modifions le fichier `movie.service.tsx`.

Commençons par la méthode `fetchMovieDetails` en ajoutant `next: { revalidate: 604800 }`

```tsx
const fetchMovieDetails = async (movieId: number): Promise<Movie> => {
  const queryParams = new URLSearchParams();
  queryParams.append('language', 'fr-FR');

  const URL = `${
    process.env.NEXT_PUBLIC_API_URL
  }/movie/${movieId}?${queryParams.toString()}`;

  const result = await fetch(`${URL}`, {
    headers: API_HEADER,
    next: { revalidate: 604800 }, // 7 days
  });
  if (!result.ok) {
    throw new Error('Failed to fetch trending movies data');
  }

  return result.json();
};
```

Et la méthode `fetchMovieReviews` en ajoutant `cache: 'no-store'`

```tsx
const fetchMovieReviews = async (movieId: number): Promise<Review[]> => {
  const URL = `${process.env.NEXT_PUBLIC_API_URL}/movie/${movieId}/reviews`;

  const result = await fetch(`${URL}`, {
    headers: API_HEADER,
    cache: 'no-store',
  });

  if (!result.ok) {
    throw new Error('Failed to get movie reviews');
  }

  const data: ReviewResponse = await result.json();
  const reviews = data.results.slice(0, 3);

  return reviews;
};
```

Maintenant, nous pouvons rajouter la gestion des critiques sur notre page de détail.
D'abord en appelant la méthode `fetchMovieReviews` :

```tsx
import Like from '@/components/like/Like';
import MovieReview from '@/components/movie/review/MovieReview';
import Note from '@/components/note/Note';
import { Review } from '@/interfaces/review.interface';
import { fetchMovieDetails, fetchMovieReviews } from '@/services/movie.service';
import Image from 'next/image';

const RSCMovieDetailsPage = async ({ params }: { params: { id: number } }) => {
  const [movie, reviews] = await Promise.all([
    fetchMovieDetails(params.id),
    fetchMovieReviews(params.id),
  ]);

  // ...

export default RSCMovieDetailsPage;
```

Et enfin, en ajoutant le composant `MovieReview` :

```tsx
import Like from '@/components/like/Like';
import MovieReview from '@/components/movie/review/MovieReview';
import Note from '@/components/note/Note';
import { Review } from '@/interfaces/review.interface';
import { fetchMovieDetails, fetchMovieReviews } from '@/services/movie.service';
import Image from 'next/image';

const RSCMovieDetailsPage = async ({ params }: { params: { id: number } }) => {
  // ...

  return (
    // ...
    <div className="movie-description">
      <div className="informations">{/*  ...  */}</div>
      <div className="reviews-list">
        {reviews.map((review: Review) => (
          <MovieReview key={review.id} review={review} />
        ))}
      </div>
    </div>
    // ...
  );
};
```

Nous pouvons redémarrer l'application avec la commande `npm run dev` et constater que les critiques des films s'affichent correctement.

### Observons ce qui se passe

Ouvrons les devtools de Chrome (ou équivalent) pour observer :

- Les chargements de fichiers Javascript
- Les appels de service REST
- La gestion du cache dans les logs

