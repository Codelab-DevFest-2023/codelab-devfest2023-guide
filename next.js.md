id:next.js

# Rendu front, action ! Découvrez les différents modes de rendu avec Next.js

## Initialisation du codelab

Duration: 5:00

Prérequis :

- [Git](https://git-scm.com/)
- [Node / npm](https://nodejs.org/) (version 18 minimum)
- [Visual Studio Code](https://code.visualstudio.com/) (ou tout autre IDE JS)

Récupérer le code du projet :

```bash
git clone https://github.com/chawax/nextjs-codelab-devfest.git
```

Installer les dépendances npm :

```bash
cd nextjs-codelab-devfest
npm install
```

## Client Side Rendering

Duration: 20:00

Pour commencer, nous allons créer un écran affichant la liste des films en rendu côté client. Nous avons donc besoin de créer :

- Un composant `MoviesList` qui affiche la liste des films
- Un hooks `useMovies` qui permet de charger la liste des films
- Une route `/movies` associée au composant `MoviesList`

TODO

La première route de notre API consiste à donner l'état de santé de notre application.

Créons le module `health` :

```bash
nest g module health
```

Créons le controller `health` :

```bash
nest g controller health
```

Dans le fichier `src\health\health.controller.ts` du controller, rajoutons la méthode `check` qui répondra aux requêtes de type `GET` :

```ts
@Get()
check(): string {
    return 'Everything is OK';
}
```

La complétion dans l'IDE nous aide à rajouter les imports de librairies nécessaires, mais au cas où voici les imports à ajouter :

```ts
import { Controller, Get } from "@nestjs/common";
```

Vérifions que tout fonctionne en lançant l'application en mode développement (live reload) :

```bash
npm run start:dev
```

La route `health` est accessible dans un navigateur via [http://localhost:3000/health](http://localhost:3000/health).

Nous allons maitenant activer le versionning d'API et la documentation Swagger.

Dans `src\main.ts`, rajoutons la configuration nécessaire dans la fonction `bootstrap`. Attention : l'instruction `app.listen` doit rester la dernière instruction de la fonction.

```ts
// VERSIONNING
app.enableVersioning({
  type: VersioningType.URI,
});

//-------- SWAGGER
const config = new DocumentBuilder()
  .setTitle("Form Earth to Moon API")
  .setDescription("A codelab to discover NestJs and more")
  .setVersion("1.0")
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup("api", app, document);
```

Les imports à utiliser sont les suivants :

```ts
import { DocumentBuilder, SwaggerModule } from "@nestjs/swagger";
```

Modifions `HealthController` pour rajouter un numéro de version sur son API :

```ts
@Controller({ path: '/health', version: '1' })
```

L'URL de la route `health` est maintenant [http://localhost:3000/v1/health](http://localhost:3000/v1/health).

L'interface Swagger est accessible via [http://localhost:3000/api](http://localhost:3000/api). Nous pouvons y tester notre API.

Duration: 10:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "step2" pour débuter cette étape : <code>git checkout -f step2</code></aside>

Nous allons maintenant créer une API pour la gestion de planètes.
Pour ce faire, nous allons créer des classes [controller](https://docs.nestjs.com/controllers), [service](https://docs.nestjs.com/providers#services), DTO et entity et placer le tout dans un module dédié.

Tout cela peut être préparé avec la commande suivante :

```bash
nest g resource planet
```

Choisissons `REST API` dans la liste proposée de couches de transport. Nous activons également la génération des points d'entrée CRUD.

Dans la classe entity `src\planet\entities\planet.entity.ts`, ajoutons les propriétés d'une planète :

```ts
name: string;

distanceToEarth: number;
```

Dans la class service `src\planet\planet.service.ts`, modifions la méthode `findAll()` pour renvoyer des données en dur :

```ts
findAll(): Planet[] {
    const planetsJSON = [
      {
        name: 'Lune',
        distanceToEarth: 384400,
      },
      {
        name: 'Venus',
        distanceToEarth: 41400000,
      },
      {
        name: 'Mars',
        distanceToEarth: 78340000,
      },
      {
        name: 'Mercure',
        distanceToEarth: 91690000,
      },
      {
        name: 'Jupiter',
        distanceToEarth: 628730000,
      },
      {
        name: 'Saturne',
        distanceToEarth: 1275000000,
      },
      {
        name: 'Uranus',
        distanceToEarth: 2723950000,
      },
      {
        name: 'Neptune',
        distanceToEarth: 4351400000,
      },
    ];

    const planets: Planet[] = Object.assign(new Array<Planet>(), planetsJSON);

    return planets;
  }
```

Nous pouvons tester la route qui liste toutes les planètes via Swagger ou via son URL : [http://localhost:3000/planet](http://localhost:3000/planet).

Faisons maintenant de même pour la ressource `starship` :

```bash
nest g resource starship
```

Ajoutons les proprietés d'un `starship` :

```ts
name: string;

speed: number;

kilometerPrice: number;
```

Modifions la méthode `findAll()` de l'entité `starship` pour renvoyer des données en dur :

```ts
findAll(): Starship[] {
    const starshipsJSON = [
      {
        name: 'Apollo',
        speed: 39000,
        kilometerPrice: 10000,
      },
      {
        name: 'SpaceX Starship',
        speed: 27000,
        kilometerPrice: 250000,
      },
      {
        name: 'Sonde Parker',
        speed: 532000,
        kilometerPrice: 50000,
      },
    ];

    const starships: Starship[] = Object.assign(new Array<Starship>(), starshipsJSON);

    return starships;
  }
```

De même que pour `planet`, la route qui liste tous les vaisseaux peut être testée via Swagger ou via son URL : [http://localhost:3000/starship](http://localhost:3000/starship).

## Server Side Rendering

Duration: 20:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "step3" pour débuter cette étape : <code>git checkout -f step3</code></aside>

Nous allons maintenant activer l'ORM [TypeORM](https://typeorm.io/) pour lire des données dans une base de données [SQLite](https://www.sqlite.org/). Une base SQLite contenant déjà des données est incluse dans le repository cloné.

Sur les classes entity, nous rajoutons l'annotation `@Entity()` pour indiquer à TypeORM de faire le mapping avec une table de la base de données.

Pour `planet` :

```ts
@Entity({ name: 'planet' })
```

Pour `starship` :

```ts
@Entity({ name: 'starship' })
```

Puis sur chaque propriété de ces 2 classes, nous rajoutons l'annotation `@Column()` pour faire le mapping avec les colonnes des tables concernées. Ex :

```ts
@Column()
name: string;
```

Les imports dans ces entités sont les suivants :

```ts
import { Column, Entity } from "typeorm";
```

Créons ensuite, dans `src\utils\default-entity.ts`, la classe `DefaultEntity` qui contient les propriétés communes à toutes les entités de notre application, à savoir :

- `id` : un identifiant technique généré automatiquement par incrément
- `uuid` : un identifiant métier unique au format UUID et généré automatiquement
- `active` : un booléen indiquant si la ressource est active

Le code de la classe `DefaultEntity` est le suivant :

```ts
// src/utils/default-entity.ts

import { Exclude } from "class-transformer";
import { Column, Generated, PrimaryGeneratedColumn } from "typeorm";

export class DefaultEntity {
  @Exclude()
  @PrimaryGeneratedColumn("identity")
  id: number;

  @Column()
  active: boolean;

  @Column({ unique: true })
  @Generated("uuid")
  uuid: string;
}
```

Enfin, faisons hériter `Planet` et `Starship` de `DefaultEntity` :

```ts
export class Planet extends DefaultEntity {
```

```ts
export class Starship extends DefaultEntity {
```

Maintenant, créons un fichier .env à la racine du projet et ajoutons y la référence à la base de données à laquelle nous souhaitons accéder :

```
SQL_MEMORY_DB_SHARED=./db/planet-starship.sqlite
```

Dans `src\app.module.ts`, dans la section `imports`, ajoutons le chargement du module TypeORM et de la base de données indiquée dans le fichier de configuration :

```ts
  imports: [
    HealthModule,
    ConfigModule.forRoot(),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: async (configService: ConfigService) => {
        return {
          type: 'sqlite',
          database: configService.get('SQL_MEMORY_DB_SHARED'),
          entities: [__dirname + '/**/*.entity{.ts,.js}'],
          synchronize: true,
        } as TypeOrmModuleOptions;
      },
    }),
    PlanetModule,
    StarshipModule,
  ],
```

Avec les imports suivants :

```ts
import { ConfigModule, ConfigService } from "@nestjs/config";
import { TypeOrmModule, TypeOrmModuleOptions } from "@nestjs/typeorm";
```

Dans `src\planet\planet.module.ts`, ajoutons l'import de TypeORM pour l'entité `Planet` et l'export du service `PlanetService` :

```ts
  imports: [TypeOrmModule.forFeature([Planet])],
  exports: [PlanetService],
```

Faisons de même pour le module starship dans `src\starship\starship.module.ts` :

```ts
  imports: [TypeOrmModule.forFeature([Starship])],
  exports: [StarshipService],
```

Dans le service `PlanetService`, ajoutons un constructeur qui injecte le repository `Planet` :

```ts
  constructor(
    @InjectRepository(Planet)
    private readonly planetRepository: Repository<Planet>,
  ) {}
```

Avec les imports suivants :

```ts
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm/repository/Repository";
```

Puis modifions la méthode `findAll()` pour utiliser le repository qui va exécuter la requête de récupération des objets `Planet` sur la base de données. La signature de la méthode est modifiée pour renvoyer un objet `Promise<Planet[]>` :

```ts
  findAll(): Promise<Planet[]> {
    return this.planetRepository.find();
  }
```

Faisons de même pour le service `StarshipService` :

```ts
  constructor(
    @InjectRepository(Starship)
    private readonly starshipRepository: Repository<Starship>,
  ) {}
```

```ts
  findAll(): Promise<Starship[]> {
    return this.starshipRepository.find();
  }
```

Les données `planet` et `starship` sont maintenant récupérées depuis la base de données. On peut le tester avec [http://localhost:3000/planet](http://localhost:3000/planet) et [http://localhost:3000/starship](http://localhost:3000/starship).

<aside class="negative">Il faut redémarrer l'application pour que les modifications des variables d'environnement soient prises en compte.</aside>

## Static Site Generation

Duration: 25:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "step4" pour débuter cette étape : <code>git checkout -f step4</code></aside>

Nous allons maintenant rajouter les opérations de récupération unitaire et d'écriture en base de données.

Modifions `src\planet\dto\create-planet.dto.ts` pour rajouter les propriétés utiles à la création d'une planète. On y ajoute des annotations utiles à l'exposition Swagger et à la validation des données :

```ts
  @ApiProperty()
  @Expose()
  @IsString()
  name: string;

  @ApiProperty()
  @Expose()
  @IsNumber()
  distanceToEarth: number;

  @ApiProperty()
  @Expose()
  @IsBoolean()
  active: boolean;
```

Avec les imports suivants :

```ts
import { ApiProperty } from "@nestjs/swagger";
import { Expose } from "class-transformer";
import { IsBoolean, IsNumber, IsString } from "class-validator";
```

`UpdatePlanetDto` hérite partiellement de `CreatePlanetDto` en rajoutant les propriétés nécessaires à la mise à jour :

```ts
  @ApiProperty()
  @Expose()
  @IsNotEmpty()
  @IsUUID()
  @IsOptional()
  uuid: string;
```

Avec les imports suivants :

```ts
import { ApiProperty, PartialType } from "@nestjs/swagger";
import { Expose } from "class-transformer";
import { IsNotEmpty, IsUUID, IsOptional } from "class-validator";
```

Procédons de même pour `CreateStarshipDto` :

```ts
  @ApiProperty()
  @Expose()
  @IsString()
  name: string;

  @ApiProperty()
  @Expose()
  @IsNumber()
  speed: number;

  @ApiProperty()
  @Expose()
  @IsNumber()
  kilometerPrice: number;

  @ApiProperty()
  @Expose()
  @IsBoolean()
  active: boolean;
```

Et pour `UpdateStarshipDto` :

```ts
  @ApiProperty()
  @Expose()
  @IsNotEmpty()
  @IsUUID()
  @IsOptional()
  uuid: string;
```

Pour que les annotations soient actives, nous devons ajouter la configuration suivante dans la méthode `bootstrap` de `src\main.ts` :

```ts
//------- IN & OUT
// Enables global behaviors on incoming DTO
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true, // Only exposed attributes will be accepted on incoming DTO
    transform: true, // Automatically converts attributes from incoming DTO when possible
    transformOptions: { enableImplicitConversion: true },
  })
);

// Enables global behaviors on outgoing entities
// For examples, @Exclude decorators will be processed
app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));
```

Avec les imports suivants :

```ts
import { ClassSerializerInterceptor, ValidationPipe } from "@nestjs/common";
```

Modifions `PlanetService` pour utiliser les DTO et le repository.

Commençons par la méthode `create()`. Le type de retour est modifié pour correspondre au type de retour de la méthode `save()` du repository :

```ts
  create(createPlanetDto: CreatePlanetDto): Promise<Planet> {
    return this.planetRepository.save(createPlanetDto);
  }
```

Remplaçons la méthode `findOne()` pour une méthode `findOneByUuid()` qui recherche une planète en fonction de son UUID :

```ts
  findOneByUuid(uuid: string): Promise<Planet | null> {
    return this.planetRepository.findOneBy({ uuid });
  }
```

La méthode `update()` est modifiée pour prendre en entrée un UUID plutôt qu'un id. Elle renvoie un objet de type `Promise<Planet>` :

```ts
  async update(uuid: string, updatePlanetDto: UpdatePlanetDto): Promise<Planet> {
    const planet = await this.findOneByUuid(uuid);

    if (!planet) {
      throw new NotFoundException();
    }

    await this.planetRepository.save({ id: planet.id, ...updatePlanetDto });

    return this.findOneByUuid(uuid);
  }
```

Idem pour la méthode `remove()` :

```ts
  async remove(uuid: string): Promise<DeleteResult> {
    const planet = await this.findOneByUuid(uuid);

    if (!planet) {
      throw new NotFoundException();
    }

    return this.planetRepository.delete({ uuid });
  }
```

Avec les imports suivants :

```ts
import { NotFoundException } from "@nestjs/common";
import { DeleteResult } from "typeorm";
```

La classe `PlanetController` doit être modifiée pour prendre en compte nos modifications sur `PlanetService` et pour utiliser des UUID :

```ts
  @Get(':uuid')
  async findOne(@Param('uuid', new ParseUUIDPipe()) uuid: string): Promise<Planet> {
    const planet = await this.planetService.findOneByUuid(uuid);

    if (planet) {
      return planet;
    }

    throw new NotFoundException();
  }
```

```ts
  @Patch(':uuid')
  update(@Param('uuid', new ParseUUIDPipe()) uuid: string, @Body() updatePlanetDto: UpdatePlanetDto): Promise<Planet> {
    return this.planetService.update(uuid, updatePlanetDto);
  }
```

```ts
  @Delete(':uuid')
  remove(@Param('uuid', new ParseUUIDPipe()) uuid: string): Promise<DeleteResult> {
    return this.planetService.remove(uuid);
  }
```

Enfin modifions les annotations de la classe pour versionner l'API et améliorer la lisibilité dans Swagger :

```ts
  @ApiTags('planets')
  @Controller({ path: '/planets', version: '1' })
  export class PlanetController {
```

Les imports à utiliser sont les suivants :

```ts
import { Body, ParseUUIDPipe } from "@nestjs/common";
import { ApiTags } from "@nestjs/swagger";
```

Procédons de même pour `StarshipService` :

```ts
  create(createStarshipDto: CreateStarshipDto): Promise<Starship> {
    return this.starshipRepository.save(createStarshipDto);
  }
```

```ts
  findOneByUuid(uuid: string): Promise<Starship | null> {
    return this.starshipRepository.findOneBy({ uuid });
  }
```

```ts
  async update(uuid: string, updateStarshipDto: UpdateStarshipDto): Promise<Starship> {
    const starship = await this.findOneByUuid(uuid);

    if (!starship) {
      throw new NotFoundException();
    }

    await this.starshipRepository.save({ id: starship.id, ...updateStarshipDto });

    return this.findOneByUuid(uuid);
  }
```

```ts
  async remove(uuid: string): Promise<DeleteResult> {
    const starship = await this.findOneByUuid(uuid);

    if (!starship) {
      throw new NotFoundException();
    }

    return this.starshipRepository.delete({ uuid });
  }
```

Et pour `StarshipController` :

```ts
  @Get(':uuid')
  findOne(@Param('uuid', new ParseUUIDPipe()) uuid: string): Promise<Starship> {
    return this.starshipService.findOneByUuid(uuid);
  }
```

```ts
  @Patch(':uuid')
  update(
    @Param('uuid', new ParseUUIDPipe()) uuid: string,
    @Body() updateStarshipDto: UpdateStarshipDto,
  ): Promise<Starship> {
    return this.starshipService.update(uuid, updateStarshipDto);
  }
```

```ts
  @Delete(':uuid')
  remove(@Param('uuid', new ParseUUIDPipe()) uuid: string): Promise<DeleteResult> {
    return this.starshipService.remove(uuid);
  }
```

```ts
@ApiTags('starships')
@Controller({ path: '/starships', version: '1' })
export class StarshipController {
```

Nous pouvons maintenant utiliser toutes les opérations CRUD sur Planet et Starship via Swagger.

Créons des planètes via la route `POST` `/v1/planets` :

```json
{
  "name": "Mercure",
  "distanceToEarth": 91690000,
  "active": true
}
```

```json
{
  "name": "Jupiter",
  "distanceToEarth": 628730000,
  "active": true
}
```

```json
{
  "name": "Saturne",
  "distanceToEarth": 1275000000,
  "active": true
}
```

```json
{
  "name": "Uranus",
  "distanceToEarth": 2723950000,
  "active": true
}
```

```json
{
  "name": "Neptune",
  "distanceToEarth": 4351400000,
  "active": true
}
```

Créons un starship via la route `POST` `/v1/starships` :

```json
{
  "name": "SpaceX Starship",
  "speed": 27000,
  "kilometerPrice": 250000,
  "active": true
}
```

## React Server Components

Duration: 15:00

<aside>Si vous n'avez pas eu le temps de finir l'étape précédente, vous pouvez faire un checkout de la branche "step5" pour débuter cette étape : <code>git checkout -f step5</code></aside>

Dans cette étape nous allons créer une nouvelle ressource : `booking` :

```bash
nest g resource booking
```

Dans `BookingModule`, ajoutons l'import des modules nécessaires :

```ts
imports: [PlanetModule, StarshipModule, TypeOrmModule.forFeature([Booking])],
```

Modifions l'entité `Booking` pour qu'elle hérite de `DefaultEntity` et déclarons la comme une entité avec mapping vers la table `booking` :

```ts
@Entity({ name: 'booking' })
export class Booking extends DefaultEntity {
```

Et ajoutons le mapping pour les propriétés suivantes :

```ts
@ManyToOne(() => Planet)
destination: Planet;

@ManyToOne(() => Starship)
starship: Starship;

@Column()
traveller: string;

@Column()
departureDate: Date;

arrivalDate: Date;

price: number;
```

Le décorateur `@ManyToOne` permet de créer un lien entre l'entité `Booking` et les entités `Planet` et `Starship`. Notons que les propriétés `arrivalDate` et `price` ne correspondent pas à des colonnes mais seront calculées (voir plus bas).

Ajoutons également les méthodes `processTravelTime()` et `processPrice()`. Elles sont appelées après le chargement d'une entité `Booking` et permettent d'alimenter les propriétés `arrivalDate` et `price`.

```ts
  @AfterLoad()
  processTravelTime() {
    if (this.destination?.distanceToEarth && this.starship?.speed) {
      const travelTime = this.destination.distanceToEarth / (this.starship.speed * 24);
      this.arrivalDate = new Date(dayjs(this.departureDate).add(travelTime, 'day').toISOString());
    }
  }

  @AfterLoad()
  processPrice() {
    if (this.destination?.distanceToEarth && this.starship?.kilometerPrice) {
      this.price = this.destination.distanceToEarth * this.starship.kilometerPrice;
    }
  }
```

Attention à l'import de `dayjs` :

```ts
import * as dayjs from "dayjs";
```

Modifions la classe `CreateBookingDto` en rajoutant les propriétés ci-après. Elle sera utilisée pour décrire l'entrée nécessaire au service de création d'une réservation.

```ts
@ApiProperty()
@Expose()
@IsBoolean()
active: boolean;

@ApiProperty()
@Expose()
@IsUUID()
destinationUuid: string;

@ApiProperty()
@Expose()
@IsUUID()
starshipUuid: string;

@ApiProperty()
@Expose()
@IsString()
traveller: string;

@ApiProperty()
@Expose()
@IsDate()
departureDate: Date;
```

Modifions également la classe `UpdateBookingDto` qui définira l'entrée nécessaire pour modifier une réservation. L'utilisation de `PartialType`, importé de `@nestjs/swagger` permet d'indiquer qu'on a les mêmes propriétés que `CreateBookingDto` mais qu'elles sont optionnelles.

```ts
export class UpdateBookingDto extends PartialType(CreateBookingDto) {
  @ApiProperty()
  @Expose()
  @IsNotEmpty()
  @IsUUID()
  @IsOptional()
  uuid: string;
}
```

Enrichir le service `BookingService`. Injectons d'abord les dépendances nécessaires dans le constructeur.

```ts
constructor(
    @InjectRepository(Booking) private readonly bookingRepository: Repository<Booking>,
    private readonly planetService: PlanetService,
    private readonly starshipService: StarshipService,
) { }
```

Modifions la méthode de création d'une réservation. Elle crée une instance de l'entité `Booking`, la complète avec les données issues du DTO et appelle le repository pour sauvegarder l'entité. On fait également appel aux services `planetService` et `starshipService` pour récupérer les entités correspondant aux uuids fournis dans le DTO.

```ts
async create(createBookingDto: CreateBookingDto): Promise<Booking> {
    const destination = await this.planetService.findOneByUuid(createBookingDto.destinationUuid);
    const starship = await this.starshipService.findOneByUuid(createBookingDto.starshipUuid);

    if (!destination || !starship) {
        throw new UnprocessableEntityException('Both destination and starship should contains existing uuids');
    }

    const booking: Booking = new Booking();
    booking.active = createBookingDto.active;
    booking.departureDate = createBookingDto.departureDate;
    booking.traveller = createBookingDto.traveller;
    booking.destination = destination;
    booking.starship = starship;

    return this.bookingRepository.save(booking);
}
```

Remplaçons la méthode `findOne` par une méthode `findOneByUuid` pour prendre en compte un UUID. La propriété `relations` permet de charger le vaisseau et la planète de destination quand on charge une réservation.

```ts
  findOneByUuid(uuid: string): Promise<Booking> {
    return this.bookingRepository.findOne({ where: { uuid }, relations: ['starship', 'destination'] });
  }
```

De même que pour `create()`, la méthode `update` est modifiée :

```ts
async update(uuid: string, updateBookingDto: UpdateBookingDto): Promise<Booking> {
    const booking = await this.findOneByUuid(uuid);

    if (!booking) {
      throw new NotFoundException();
    }

    if (updateBookingDto.destinationUuid) {
      const destination = await this.planetService.findOneByUuid(updateBookingDto.destinationUuid);
      if (!destination) {
        throw new UnprocessableEntityException('The provided destination UUID doesn\'t map to an existing destination');
      }
      booking.destination = destination;
    }

    if (updateBookingDto.starshipUuid) {
      const starship = await this.starshipService.findOneByUuid(updateBookingDto.starshipUuid);
      if (!starship) {
        throw new UnprocessableEntityException('The provided starship UUID doesn\'t map to an existing starship');
      }
      booking.starship = starship;
    }

    booking.active = updateBookingDto.active;
    booking.departureDate = updateBookingDto.departureDate;
    booking.traveller = updateBookingDto.traveller;

    return this.bookingRepository.save(booking);
}
```

Compléter les méthodes `remove` et `findAll` :

```ts
remove(uuid: string): Promise<DeleteResult> {
    return this.bookingRepository.delete({ uuid });
}
```

```ts
findAll(): Promise<Booking[]> {
    return this.bookingRepository.find({ relations: ['starship', 'destination'] });
}
```

Et enfin compléter le contrôleur `BookingController` :

```ts
@Post()
create(@Body() createBookingDto: CreateBookingDto): Promise<Booking> {
    return this.bookingService.create(createBookingDto);
}

@Patch(':uuid')
update(
    @Param('uuid', new ParseUUIDPipe()) uuid: string,
    @Body() updateBookingDto: UpdateBookingDto,
): Promise<Booking> {
    return this.bookingService.update(uuid, updateBookingDto);
}

@Delete(':uuid')
remove(@Param('uuid', new ParseUUIDPipe()) uuid: string): Promise<DeleteResult> {
    return this.bookingService.remove(uuid);
}

@Get()
findAll(): Promise<Booking[]> {
    return this.bookingService.findAll();
}

@Get(':uuid')
findOne(@Param('uuid', new ParseUUIDPipe()) uuid: string): Promise<Booking> {
    return this.bookingService.findOneByUuid(uuid);
}
```

```ts
@ApiTags('bookings')
@Controller({ path: '/bookings', version: '1' })
```

Il est maintenant possible de manipuler les bookings avec leur API dans Swagger.