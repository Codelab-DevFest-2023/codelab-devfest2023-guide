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

Cloner le projet pour l'exercice sur le client side rendering, en se positionnant sur la branche `start` :

```bash
git clone --branch start https://github.com/Codelab-DevFest-2023/codelab-devfest2023-csr.git
cd codelab-devfest2023-csr
npm install
```

Lancer la commande `npm run dev` et vérifier qu'on a bien une application qui s'affiche sur l'url `http://localhost:5173`.

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

Pour cette partie nous travaillerons dans le répertoire `codelab-devfest2023-csr`.

<aside>Pour se concentrer sur ce qui est spécifique au client side rendering, certains éléments sont déjà fournis dans le repository : carte d'un film, méthodes de fetch des données, composant de layout, etc...</aside>

### Création d'un fichier d'environnement

Pour pouvoir utiliser l'API TMDB nous devons fournir une clé d'API dans un fichier `.env`. Créer ce fichier `.env` à la racine du projet avec le contenu suivant :

```
VITE_API_URL=https://api.themoviedb.org/3
VITE_API_KEY=eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIyZmMxMDQ0YjI0ZDdkYjcyY2RlZmJmNTBkNTkyNzhiYyIsInN1YiI6IjY0YzU2ZTgyNjNhNjk1MDEwMzk5Y2I0YiIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.Rc42SZEbnqH6PvJRj1GAVYTADLcR5vNzArE1P333_dI
```

En utilisant une clé d'API parmi celles-ci :
- `eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiIyZmMxMDQ0YjI0ZDdkYjcyY2RlZmJmNTBkNTkyNzhiYyIsInN1YiI6IjY0YzU2ZTgyNjNhNjk1MDEwMzk5Y2I0YiIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.Rc42SZEbnqH6PvJRj1GAVYTADLcR5vNzArE1P333_dI`
- clé 2
- clé 3
- clé 4
- clé 5

### Liste des films

Pour commencer, nous allons créer un écran affichant la liste des films. Nous avons donc besoin de créer :

- Un hook `useMovies` qui permet de charger la liste des films
- Une page `MoviesPage` qui affiche la liste des films
- Une route `/movies` associée au composant `MoviesPage`

Dans le répertoire `/src/hooks` du projet, créons le fichier `movies.ts` et ajoutons y le hook `useMovies`. Ce hook s'appuie sur la librairie React Query pour gérer du cache.

```ts
import { useQuery } from '@tanstack/react-query';
import { AxiosError } from 'axios';
import { Movie } from '../interfaces/movie.interface';
import { getMovies } from '../services/movie.service';

const useMovies = () => {
  return useQuery<Movie[], AxiosError>({
    queryKey: ['movies'],
    queryFn: () => getMovies(),
  });
};

export { useMovies };
```

Puis dans le répertoire `/src/pages` nous allons modifier le fichier `MoviesPage.tsx` :
- Appel du hook `useMovies` pour récupérer les films (ce hook s'appuie sur la librairie React Query)
- Affichage de `Chargement ...` pendant le chargement des données
- Affichage d'un message d'erreur et d'un bouton `Réessayer`
- Affichage d'une carte pour chaque film chargé
- Affichage d'un message dans le cas où on n'a aucun film

Le composant doit ressembler à ça :

```ts
import MovieCard from '../../components/movie/card/MovieCard';
import { useMovies } from '../../hooks/movies';

const MoviesPage = () => {
  const { data: movies, isFetching, isError, isFetched, refetch } = useMovies();
  return (
    <div className="lg:mx-44 mx-4 space-y-4 lg:pt-6 pt-14 pb-20">
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
      {
        isFetched && movies && movies?.length > 0 && (
        <ul className="movies-list grid xl:grid-cols-4 lg:grid-cols-3 sm:grid-cols-2 grid-cols-1 gap-6">
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

export default MoviesPage;
```

Enfin nous allons ajouter une route `/movies` dans le fichier `/src/App.tsx` et l'associer au composant `MoviesPage` comme ci-dessous.

```ts
import { BrowserRouter, Route, Routes } from 'react-router-dom';
import Layout from './components/layout/Layout';
import MoviesPage from './pages/movies/MoviesPage';
import HomePage from './pages/home/HomePage';

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<HomePage />} />
          <Route path="movies" element={<MoviesPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
};

export default App;
```

Il ne reste plus qu'a rajouter un accès vers notre nouvelle route dans le fichier `/src/components/header/Header.tsx`

```ts
import popcornLogo from '../../assets/popcorn.jpg';

const Header = () => {
  return (
    <nav className="w-full bg-primary z-50 fixed top-0 left-0 right-0">
      <div className="flex items-center bg-primary lg:mx-44 lg:justify-between lg:py-5  lg:gap-0 gap-6 mx-4 py-2">
        <a href="/" className="flex gap-4 items-center -m-1.5 p-1.5">
          <img src={popcornLogo} alt="Logo popcorn" width={40} height={40} />
          <span className="hidden sm:block text-white font-semibold leading-6 xl:text-lg text-base">
            Rendu front, action !
          </span>
        </a>

        <ul className="text-white grid lg:grid-cols-3 sm:grid-cols-2 grid-cols-1 gap-4">
          <li>
            <a
              href="/movies"
              className="flex items-center gap-2 font-semibold leading-6 xl:text-lg text-base"
            >
              Liste des films
            </a>
          </li>
        </ul>

        <img
          className="lg:block hidden"
          src={popcornLogo}
          alt="Logo popcorn"
          width={40}
          height={40}
        />
      </div>
    </nav>
  );
};

export default Header;
```

Nous pouvons démarrer l'application avec la commande `npm run dev` et constater que la liste des films s'affiche correctement `http://localhost:5173/movies`.

### Détail d'un film

Ensuite nous allons créer un écran affichant le détail de chaque film. Nous avons besoin de créer :

- Un hook `useMovie` qui permet de charger le détail du film
- Un composant `MoviePage` qui définit la page de détail d'un film
- Une route `/movies/:id` qui définit des routes pour le

Dans le répertoire `/src/hooks` du projet, modifions le fichier `movie.hook.ts`. avec le contenu suivant :

```ts
import { useQuery } from '@tanstack/react-query';
import { AxiosError } from 'axios';
import { Movie } from '../interfaces/movie.interface';
import { getMovieDetails, getMovies } from '../services/movie.service';

// ...

const useMovie = (movieId: number) => {
  return useQuery<Movie, AxiosError>({
    queryKey: ['movies', movieId],
    queryFn: () => getMovieDetails(movieId),
  });
};

export { useMovie, useMovies };
```

Puis dans le répertoire `/src/pages` nous allons modifier le fichier `MoviePage.tsx` :
- Utilisation du hook `useMovie` pour récupérer le détail du film (le hook utilise React Query)
- Utilisation des statuts et méthodes retournés par React Query pour gérer un message de chargement et gérer les erreurs de chargement
- Affichage des informations sur le film

```ts
import { useParams } from 'react-router-dom';
import { useMovie } from '../../hooks/movies';
import Like from '../../components/like/Like';
import Note from '../../components/note/Note';

const MoviePage = () => {
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
    <div className="flex md:flex-row flex-col">
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
          <div className="poster z-10 md:order-first order-last">
            <img
              src={posterUrl}
              alt={movie.title}
              className="aspect-[2/3] object-cover h-full"
              height={750}
              width={500}
              sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            />
          </div>
          <div className="description relative w-full">
            <img
              className="absolute z-0 object-cover h-full w-[inherit] brightness-50"
              alt={movie.title}
              src={backdropPathUrl}
            />
            <div className="relative flex flex-col">
              <div className="flex flex-col gap-3 ml-4 text-white mt-3">
                <h1 className="text-xl font-semibold">{movie.title}</h1>
                <div className="flex gap-2">
                  {movie.genres.map((genre) => {
                    return <p key={genre.id}>{genre.name}</p>;
                  })}
                </div>
                <p>{movie.tagline}</p>
                <p className="mt-2 mr-10">{movie.overview}</p>
                <div className="flex items-center gap-3">
                  <Note note={movie.vote_average} />
                  <Like id={movie.id} />
                </div>
              </div>
            </div>
          </div>
        </>
      )}
    </div>
  );
};

export default MoviePage;
```

Enfin nous allons créer une route `/movies/:id` dans le fichier `/src/App.tsx`.

```ts
import { BrowserRouter, Route, Routes } from 'react-router-dom';
import Layout from './components/layout/Layout';
import MoviePage from './pages/movie/MoviePage';
import MoviesPage from './pages/movies/MoviesPage';
import HomePage from './pages/home/HomePage';

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<HomePage />} />
          <Route path="movies" element={<MoviesPage />} />
          <Route path="movie/:movieId" element={<MoviePage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
};

export default App;
```

Nous pouvons démarrer l'application avec la commande `npm run dev`, cliquer sur un film afin d'accéder à la nouvelle page de détail.

### Analysons cette application

Ouvrons les devtools de Chrome (ou équivalent) pour observer :

- Les chargements de fichiers Javascript
- Les appels de service REST

Enfin affichons le code source de la page pour constater que cette page n'est pas très "SEO friendly" !

## Server Side Rendering

Duration: 35:00

Dans cette deuxième partie, nous allons développer une application en Server Side Rendering s'appuyant sur Next.js et Tailwind.

<aside>Pour se concentrer sur ce qui est spécifique au server side rendering, certains composants vous sont déjà fournis dans le repository : carte d'un film, méthodes de fetch, composant de layout, etc...</aside>

<aside class="negative">Cet exercice utilisera le routage <strong>pages</strong> (Next.js < 13).</aside>

### Liste des films

Nous allons créer un écran affichant la liste des films. Nous aurons besoin de créer la page d'affichage de la liste des films.

Dans le répertoire `/src/pages/ssr` du projet, modifions le fichier `index.tsx`.

Nous allons utiliser la méthode `getServerSideProps` qui permet de récupérer des données côté serveur.
Ici, la méthode `getServerSideProps` récupère des données l'API et les transmet en tant que propriété `movies` à la page `SSRPage`. Ces données sont disponibles côté serveur lors de la demande de la page.

```tsx
import { Movie } from '@/interfaces/movie.interface';
import { getMovies } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';

const SSRPage = ({
  movies,
}: InferGetServerSidePropsType<typeof getServerSideProps>) => {
  // ...
};

export const getServerSideProps: GetServerSideProps<{
  movies: Movie[];
}> = async () => {
  const { results: movies } = await getMovies();
  return { props: { movies } };
};

export default SSRPage;
```

<aside>Vous pouvez définir <strong>getServerSideProps</strong> pour chaque page individuelle, ce qui permet d'obtenir des données spécifiques à cette page.</aside>

Il ne reste plus qu'a finir de construire notre page, en affichant la liste des films :

```tsx
import MovieCard from '@/components/movie/card/MovieCard';
import { Movie } from '@/interfaces/movie.interface';
import { getMovies } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';

const SSRPage = ({
  movies,
  total,
}: InferGetServerSidePropsType<typeof getServerSideProps>) => {
  const pathname = '/ssr';
  return (
    <>
      <Head>
        <title>Server Side Rendering</title>
      </Head>
      <main className="lg:mx-44 mx-4 space-y-4 lg:pt-6 pt-14 pb-20">
        <ul className="movies-list grid xl:grid-cols-4 lg:grid-cols-3 sm:grid-cols-2 grid-cols-1 gap-6">
          {movies?.map((movie: Movie) => (
            <li key={movie.id}>
              <MovieCard movie={movie} pathname={pathname} />
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

Afin d'optimiser l'affichage des images sur l'application. Nous allons utiliser le composant Image de Next.js.

Le composant Image est conçu pour améliorer les performances et l'expérience utilisateur en matière de gestion des images, en simplifiant l'optimisation, le chargement asynchrone et l'adaptation des images aux besoins de l'utilisateur, tout en réduisant la charge de travail pour les développeurs.

Il offre plusieurs avantages par rapport à l'utilisation d'une balise HTML `img` standard

Dans le répertoire `/src/components/movie/card` du projet, ouvrons le fichier `MovieCard.tsx`.

Remplaçons le composant `img` par un composant `Image` :

```tsx
import Image from 'next/image';

// ...

<Image
  src={posterUrl}
  alt={movie.title}
  className="rounded-t-lg aspect-[2/3] object-cover"
  height={750}
  width={500}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  priority
/>;

// ...
```

### Recherche de films

Nous allons créer un composant qui nous permettra de rechercher un film.
Dans le cas d'une recherche côté serveur, la recherche de l'utilisateur sera envoyé dans l'url de la page en paramètre `http://localhost:3000/ssr?query=xxxx`.

Dans le répertoire `/src/components/search` du projet, ouvrons le fichier `SearchBox.tsx`.

<aside class="negative">Ce composant sera composé de plusieurs hooks React et devra s'exécuter côté client.Pour cela, il est impératif de spécifier <strong>'use client'</strong> dans le fichier.</aside>

```tsx
'use client'; // Composant client
import { QUERY_PARAMS } from '@/constants';
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

    const queryParams = new URLSearchParams();
    queryParams.append(QUERY_PARAMS.QUERY, encodeURI(value));

    if (value.length > 3) {
      router.push(`${currentPathname}?${queryParams.toString()}`);
    }

    if (value.length <= 2 && searchParams?.get(QUERY_PARAMS.QUERY)) {
      router.push(currentPathname);
    }
  };

  return (
    <input
      type="text"
      className="text-xl py-3 px-6 bg-white rounded-full w-fit focus-visible:ring-primary focus-visible:ring-offset-primary"
      placeholder="Recherche ..."
      value={searchValue}
      onChange={handleSearchChange}
    />
  );
};

export default SearchBox;
```

Avant le rendu, la page analysera l'url afin :

- D'appeler la route `search/movie` en cas de presence du paramètre `query`
- D'appeler la route `movie/popular` en cas d'absence du paramètre `query`

Il faut alors modifier la méthode `getServerSideProps` de la page `/src/pages/ssr/index.tsx` afin de gérer ces deux cas :

```tsx
import MovieCard from '@/components/movie/card/MovieCard';
import { transformParsedUrlQuery } from '@/helpers';
import { Movie } from '@/interfaces/movie.interface';
import { fetchMovies } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';

// ...

export const getServerSideProps: GetServerSideProps<{
  movies: Movie[];
}> = async ({ query: params }) => {
  const searchParams = transformParsedUrlQuery(params);
  const { results: movies } = await fetchMovies(searchParams);
  return { props: { movies } };
};
```

Il ne reste plus qu'a ajouter le composant `Search.tsx`

```tsx
import MovieCard from '@/components/movie/card/MovieCard';
import SearchBox from '@/components/search/SearchBox';
import { transformParsedUrlQuery } from '@/helpers';
import { Movie } from '@/interfaces/movie.interface';
import { fetchMovies } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';

// ...
<>
  <Head>
    <title>Server Side Rendering</title>
  </Head>
  <main className="lg:mx-44 mx-4 space-y-4 lg:pt-6 pt-14 pb-20">
    <SearchBox />
    <ul className="movies-list grid xl:grid-cols-4 lg:grid-cols-3 sm:grid-cols-2 grid-cols-1 gap-6">
      {movies?.map((movie: Movie) => (
        <li key={movie.id}>
          <MovieCard movie={movie} pathname={pathname} />
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

### Détail d'un film

Ensuite, nous allons créer un écran affichant le détail de chaque film. Nous aurons besoin de créer la page d'affichage des détails d'un film.

Dans le répertoire `/src/pages/ssr` du projet, modifions le fichier `[id].tsx`.

Nous allons réutiliser la méthode `getServerSideProps` pour récupérer les données concernant le film passé en paramètre.

```tsx
import { Movie } from '@/interfaces/movie.interface';
import { getMovieDetails } from '@/services/movie.service';
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
    const movie = await getMovieDetails(id);
    return { props: { movie } };
  } else {
    throw new Error('Missing id parameter');
  }
};

export default SSRMovieDetailsPage;
```

Il ne reste plus qu'a finir de construire notre page, En affichant :

- L'affiche du film
- Le titre du film
- Le synopsis du film
- La note du film

```tsx
import Note from '@/components/note/Note';
import { Movie } from '@/interfaces/movie.interface';
import { getMovieDetails } from '@/services/movie.service';
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
      <div className="flex md:flex-row flex-col">
        <div className="poster z-10 md:order-first order-last">
          <Image
            src={posterUrl}
            alt={movie.title}
            className="aspect-[2/3] object-cover h-full"
            height={750}
            width={500}
            sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            priority
          />
        </div>
        <div className="description relative w-full">
          <Image
            className="block z-0 object-cover brightness-50"
            alt={movie.title}
            src={backdropPathUrl}
            fill
            priority
          />
          <div className="relative flex flex-col">
            <div className="flex flex-col gap-3 ml-4 text-white mt-3">
              <h1 className="text-xl font-semibold">{movie.title}</h1>
              <div className="flex gap-2">
                {movie.genres.map((genre) => {
                  return <p key={genre.id}>{genre.name}</p>;
                })}
              </div>
              <p>{movie.tagline}</p>
              <p className="mt-2 mr-10">{movie.overview}</p>
              <div className="flex items-center gap-3">
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

### Gestion de favoris

Maintenant, mettons en place une gestion des favoris en utilisant le `localstorage` du navigateur. Nous allons utiliser un nouveau composant `Like`.

<aside class="negative">Ce composant aura besoin d'accèder au <strong>localstorage</strong> du navigateur. Pour cela, il doit s'exécuter côté client (<strong>'use client'</strong>).</aside>

Dans le répertoire `/src/components/like` du projet, ouvrons le fichier `Like.tsx`.

```tsx
'use client';
import { useEffect, useState } from 'react';
import './like.module.css';

interface Props {
  id: number;
}

const Like = ({ id }: Props) => {
  const [filled, setFilled] = useState(false);

  useEffect(() => {
    const favorites = localStorage.getItem('favorites');
    if (favorites) {
      const isFind = JSON.parse(favorites).find(
        (element: { id: number }) => element.id === id
      );
      if (isFind) {
        setFilled(true);
      }
    } else {
      setFilled(false);
    }
  }, [id]);

  const handleClick = () => {
    const favorites = localStorage.getItem('favorites');
    if (favorites) {
      const favoritesList = JSON.parse(favorites);

      const filterFav = favoritesList.filter(
        (fav: { id: number }) => fav.id !== id
      );

      if (filterFav.length === favoritesList.length) {
        setFilled(true);
        favoritesList.push({ id: id });
        localStorage.setItem('favorites', JSON.stringify(favoritesList));
      } else {
        setFilled(false);
        localStorage.setItem('favorites', JSON.stringify(filterFav));
      }
    } else {
      setFilled(true);
      localStorage.setItem('favorites', JSON.stringify([{ id: id }]));
    }
  };

  return (
    <button
      className={`heart-button ${filled ? 'filled' : ''}`}
      onClick={handleClick}
    >
      <svg className="heart-icon" viewBox="0 0 26 26">
        <path d="M12 21.35l-1.45-1.32C5.4 16.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C15.09 3.81 16.76 3 18.5 3 21.58 3 24 5.42 24 8.5c0 3.78-3.4 7.86-8.55 11.54L12 21.35z" />
      </svg>
    </button>
  );
};

export default Like;
```

Il ne reste plus qu'a rajouter ce nouveau composant dans la page SSR (`/src/pages/ssr/[id].tsx`) :

```tsx
import Like from '@/components/like/Like';
import Note from '@/components/note/Note';
import { Movie } from '@/interfaces/movie.interface';
import { getMovieDetails } from '@/services/movie.service';
import { GetServerSideProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';
import Image from 'next/image';

// ...
<div className="relative flex flex-col">
  <div className="flex flex-col gap-3 ml-4 text-white mt-3">
    <h1 className="text-xl font-semibold">{movie.title}</h1>
    <div className="flex gap-2">
      {movie.genres.map((genre) => {
        return <p key={genre.id}>{genre.name}</p>;
      })}
    </div>
    <p>{movie.tagline}</p>
    <p className="mt-2 mr-10">{movie.overview}</p>
    <div className="flex items-center gap-3">
      <Note note={movie.vote_average} />
      <Like id={movie.id} />
    </div>
  </div>
</div>;
// ...
```

### Observons ce qui se passe

Ouvrons les devtools de Chrome (ou équivalent) pour observer :

- Les chargements de fichiers Javascript
- Les appels de service REST

## Static Site Generation

Duration: 10:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "ssr-end" pour débuter cette étape : <code>git checkout -f ssr-end</code></aside>

Dans cette troisième partie, nous allons développer la même application que précédement mais en Static Site Generation en s'appuyant sur Next.js et Tailwind.

<aside class="negative">Cet exercice utilisera le routage <strong>pages</strong> (Next.js < 13).</aside>

### Liste des films

Comme dans l'exercice précédent, nous aurons besoin de créer la page d'affichage de la liste des films.

Dans le répertoire `/src/pages/ssg` du projet, modifions le fichier `index.tsx`.

Nous allons utiliser la méthode `getStaticProps` qui permet de générer des pages statiques lors de la construction de l'application, en précalculant les données à afficher sur ces pages.

Ici, la méthode `getStaticProps` récupère des données l'API et les transmet en tant que propriété `movies` à la page `SSRPage`. Les données sont précalculées au moment de la construction de l'application et servies en tant que pages statiques.

```tsx
import { Movie } from '@/interfaces/movie.interface';
import { getMovies } from '@/services/movie.service';
import { GetStaticProps, InferGetServerSidePropsType } from 'next';

const SSGPage = ({
  movies,
}: InferGetServerSidePropsType<typeof getStaticProps>) => {
  // ...
};

export const getStaticProps: GetStaticProps<{
  movies: Movie[];
}> = async () => {
  const { results: movies } = await getMovies();
  return { props: { movies } };
};

export default SSGPage;
```

Il ne reste plus qu'a finir de construire notre page, en affichant la liste des films :

```tsx
import MovieCard from '@/components/movie/card/MovieCard';
import { Movie } from '@/interfaces/movie.interface';
import { getMovies } from '@/services/movie.service';
import { GetStaticProps, InferGetServerSidePropsType } from 'next';
import Head from 'next/head';

const SSGPage = ({
  movies,
}: InferGetServerSidePropsType<typeof getStaticProps>) => {
  const pathname = '/ssg';
  return (
    <>
      <Head>
        <title>Static Site Generation</title>
      </Head>
      <main className="lg:mx-44 mx-4 space-y-4 lg:pt-6 pt-14 pb-20">
        <ul className="movies-list grid xl:grid-cols-4 lg:grid-cols-3 sm:grid-cols-2 grid-cols-1 gap-6">
          {movies?.map((movie: Movie) => (
            <li key={movie.id}>
              <MovieCard movie={movie} pathname={pathname} />
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

Nous pouvons démarrer l'application avec la commande `npm run dev` et constater que la liste des films s'affiche correctement à l'adresse `http://localhost:3000/ssg`.

### Détail d'un film

Ensuite, nous allons créer un écran affichant le détail de chaque film.

Dans le répertoire `/src/pages/ssg` du projet, modifions le fichier `[id].tsx`.

Nous allons réutiliser la méthode `getStaticProps` pour récupérer les données concernant le film passé en paramètre.

```tsx
import { Movie } from '@/interfaces/movie.interface';
import { getMovieDetails } from '@/services/movie.service';
import { GetStaticProps, InferGetServerSidePropsType } from 'next';

const SSGMovieDetailsPage = ({
  movie,
}: InferGetServerSidePropsType<typeof getStaticProps>) => {
  // ...
};

export const getStaticProps: GetStaticProps<{
  movie: Movie;
}> = async ({ params }) => {
  if (params?.id) {
    const id = Number(params.id);
    const movie = await getMovieDetails(id);
    return { props: { movie } };
  } else {
    throw new Error('Missing id parameter');
  }
};

export default SSGMovieDetailsPage;
```

Nous devons aussi utiliser la méthode `getStaticPaths` afin de générer les chemins (URL) pour les pages dynamiques lors de la génération statique.

<aside><strong>getStaticPaths</strong> est souvent utilisée avec <strong>getStaticProps</strong> pour générer des pages dynamiques précalculées, où les chemins sont basés sur des données provenant d'une source externe, comme une API ou une base de données.</aside>

```tsx
import { Movie } from '@/interfaces/movie.interface';
import { getMovieDetails, getMovies } from '@/services/movie.service';
import {
  GetStaticPaths,
  GetStaticProps,
  InferGetServerSidePropsType,
} from 'next';

const SSGMovieDetailsPage = ({
  movie,
}: InferGetServerSidePropsType<typeof getStaticProps>) => {
  // ...
};

export const getStaticProps: GetStaticProps<{
  movie: Movie;
}> = async ({ params }) => {
  if (params?.id) {
    const id = Number(params.id);
    const movie = await getMovieDetails(id);
    return { props: { movie } };
  } else {
    throw new Error('Missing id parameter');
  }
};

export const getStaticPaths: GetStaticPaths = async () => {
  const { results: movies } = await getMovies();
  return {
    paths: movies.map((movie) => `/ssg/${movie.id}`),
    fallback: false,
  };
};

export default SSGMovieDetailsPage;
```

Il ne reste plus qu'a finir de construire notre page :

```tsx
import Like from '@/components/like/Like';
import Note from '@/components/note/Note';
import { Movie } from '@/interfaces/movie.interface';
import { getMovieDetails, getMovies } from '@/services/movie.service';
import {
  GetStaticPaths,
  GetStaticProps,
  InferGetServerSidePropsType,
} from 'next';
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
      <div className="flex md:flex-row flex-col">
        <div className="poster z-10 md:order-first order-last">
          <Image
            src={posterUrl}
            alt={movie.title}
            className="aspect-[2/3] object-cover h-full"
            height={750}
            width={500}
            sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            priority
          />
        </div>
        <div className="description relative w-full">
          <Image
            className="block z-0 object-cover brightness-50"
            alt={movie.title}
            src={backdropPathUrl}
            fill
            priority
          />
          <div className="relative flex flex-col">
            <div className="flex flex-col gap-3 ml-4 text-white mt-3">
              <h1 className="text-xl font-semibold">{movie.title}</h1>
              <div className="flex gap-2">
                {movie.genres.map((genre) => {
                  return <p key={genre.id}>{genre.name}</p>;
                })}
              </div>
              <p>{movie.tagline}</p>
              <p className="mt-2 mr-10">{movie.overview}</p>
              <div className="flex items-center gap-3">
                <Note note={movie.vote_average} />
                <Like id={movie.id} />
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

### Observons ce qui se passe

Ouvrons les devtools de Chrome (ou équivalent) pour observer :

- Les chargements de fichiers Javascript
- Les appels de service REST

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
import { fetchMovies } from '@/services/movie.service';

export const revalidate = 0;

interface Props {
  searchParams: { [key: string]: string | string[] | undefined };
}

const RSCPage = async ({ searchParams }: Props) => {
  const pathname = '/rsc';
  const { results: movies } = await fetchMovies(searchParams);

  return (
    <main className="lg:mx-44 mx-4 space-y-4 lg:pt-6 pt-14 pb-20">
      <SearchBox />
      <ul className="movies-list grid xl:grid-cols-4 lg:grid-cols-3 sm:grid-cols-2 grid-cols-1 gap-6">
        {movies?.map((movie: Movie) => (
          <li key={movie.id}>
            <MovieCard movie={movie} pathname={pathname} />
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

Nous pouvons démarrer l'application avec la commande `npm run dev` et constater que la liste des films s'affiche correctement à l'adresse `http://localhost:3000/rsc`.

### Détail d'un film

Ensuite, nous allons créer un écran affichant le détail de chaque film.

Dans le répertoire `/src/app/rsc/[id]` du projet, modifions le fichier `page.tsx`.

Comme précédement, nous pouvons directement faire un appel API dans notre composant serveur :

```tsx
import Like from '@/components/like/Like';
import Note from '@/components/note/Note';
import { getMovieDetails } from '@/services/movie.service';
import Image from 'next/image';

const RSCMovieDetailsPage = async ({ params }: { params: { id: number } }) => {
  const movie = await getMovieDetails(params.id);

  const posterUrl = `https://image.tmdb.org/t/p/w500/${movie.poster_path}`;
  const backdropPathUrl = `https://image.tmdb.org/t/p/original/${movie.backdrop_path}`;

  return (
    <main className="flex md:flex-row flex-col">
      <div className="poster z-10 md:order-first order-last">
        <Image
          src={posterUrl}
          alt={movie.title}
          className="aspect-[2/3] object-cover h-full"
          height={750}
          width={500}
          sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
          priority
        />
      </div>
      <div className="description relative w-full">
        <Image
          className="block z-0 object-cover brightness-50"
          alt={movie.title}
          src={backdropPathUrl}
          fill
          priority
        />
        <div className="relative flex flex-col">
          <div className="flex flex-col gap-3 ml-4 text-white mt-3">
            <h1 className="text-xl font-semibold">{movie.title}</h1>
            <div className="flex gap-2">
              {movie.genres.map((genre) => {
                return <p key={genre.id}>{genre.name}</p>;
              })}
            </div>
            <p>{movie.tagline}</p>
            <p className="mt-2 mr-10">{movie.overview}</p>
            <div className="flex items-center gap-3">
              <Note note={movie.vote_average} />
              <Like id={movie.id} />
            </div>
          </div>
        </div>
      </div>
    </main>
  );
};

export default RSCMovieDetailsPage;
```

### Affichage des critiques de films

Afin de tirer partie de la force des React Server Components, nous allons afficher les critiques du film sur la page de détail.
Nous allons :

- Mettre en cache la requête de récupération du film (7 jours)
- Ne pas mettre de cache sur la requête de récupération des critiques
- Garder nos composants client

Dans le répertoire `/src/services` du projet, modifions le fichier `movie.service.tsx`.

Commençons par la méthode `getMovieDetails` en ajoutant `next: { revalidate: 604800 }`

```tsx
const getMovieDetails = async (movieId: number): Promise<Movie> => {
  const queryParams = new URLSearchParams();
  queryParams.append(QUERY_PARAMS.LANGUAGE, DEFAULT_PARAMS.LANGUAGE);

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

Et la méthode `getMovieReviews` en ajoutant `cache: 'no-store'`

```tsx
const getMovieReviews = async (movieId: number): Promise<Review[]> => {
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

Maintenant, nous pouvons rajouter la gestion des critiques sur notre page de détail :

```tsx
import Like from '@/components/like/Like';
import MovieReview from '@/components/movie/review/MovieReview';
import Note from '@/components/note/Note';
import { Review } from '@/interfaces/review.interface';
import { getMovieDetails, getMovieReviews } from '@/services/movie.service';
import Image from 'next/image';

const RSCMovieDetailsPage = async ({ params }: { params: { id: number } }) => {
  const [movie, reviews] = await Promise.all([
    getMovieDetails(params.id),
    getMovieReviews(params.id),
  ]);

  const posterUrl = `https://image.tmdb.org/t/p/w500/${movie.poster_path}`;
  const backdropPathUrl = `https://image.tmdb.org/t/p/original/${movie.backdrop_path}`;

  return (
    // ...
    <div className="relative flex flex-col">
      <div className="flex flex-col gap-3 ml-4 text-white mt-3">
        {/*  ...  */}
      </div>
      <div className="grid lg:grid-cols-3 sm:grid-cols-2 grid-cols-1 gap-6 p-4">
        {reviews.map((review: Review) => (
          <MovieReview key={review.id} review={review} />
        ))}
      </div>
    </div>
    // ...
  );
};

export default RSCMovieDetailsPage;
```

### Observons ce qui se passe

Ouvrons les devtools de Chrome (ou équivalent) pour observer :

- Les chargements de fichiers Javascript
- Les appels de service REST

