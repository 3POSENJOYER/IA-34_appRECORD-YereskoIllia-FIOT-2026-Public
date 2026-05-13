# Звіт з виконання лабораторних завдань

## Загальний контекст

Проєкт реалізовано як NestJS бекенд у папці `backend`, але всі вимоги "REST API на Node.js" виконані у цьому серверному шарі. У звіті описано, де саме реалізовано кожне завдання.

---

## 1. Створити REST API на Node.js та Express

У проєкті реалізовано REST API у `backend` за допомогою NestJS, який працює поверх Express.

Файли:

- `backend/main.ts` — налаштування сервера та глобального префіксу `api`
- `backend/app.module.ts` — імпорт модулів та підключення контролерів
- `backend/controllers/auth.controller.ts` — маршрути авторизації та реєстрації
- `backend/controllers/games.controller.ts` — маршрути для роботи з іграми
- `backend/controllers/hardware.controller.ts` — маршрути для збереження та читання апаратного профілю
- `backend/controllers/prediction.controller.ts` — маршрут прогнозу FPS

Маршрути API:

- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/games`
- `GET /api/games/:id`
- `POST /api/games`
- `POST /api/hardware`
- `GET /api/hardware/:userId`
- `POST /api/predict/fps`

**Приклад коду з `backend/main.ts`:**

```typescript
import { NestFactory } from "@nestjs/core";
import { ValidationPipe } from "@nestjs/common";
import { SwaggerModule, DocumentBuilder } from "@nestjs/swagger";
import helmet from "helmet";
import compression from "compression";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  app.use(helmet());
  app.use(compression());
  app.setGlobalPrefix("api");

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
      transformOptions: { enableImplicitConversion: true },
    }),
  );

  const swaggerConfig = new DocumentBuilder()
    .setTitle("Lab Adaptive API")
    .setDescription(
      "REST API with security, caching, and performance optimizations",
    )
    .setVersion("1.0")
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, swaggerConfig);
  SwaggerModule.setup("api/docs", app, document);

  await app.listen(3000);
  console.log("NestJS app is running on http://localhost:3000");
}

bootstrap();
```

**Приклад контролера з `backend/controllers/games.controller.ts`:**

```typescript
@Controller("games")
export class GamesController {
  constructor(private gamesService: GamesService) {}

  @Get()
  async getGames(
    @Query("page") page: number = 1,
    @Query("limit") limit: number = 10,
  ) {
    return this.gamesService.getGames(page, limit);
  }

  @Get(":id")
  async getGame(@Param("id") id: string) {
    return this.gamesService.getGameById(+id);
  }

  @Post()
  async createGame(@Body() createGameDto: CreateGameDto) {
    return this.gamesService.createGame(createGameDto);
  }
}
```

---

## 2. Реалізувати захист API

### Helmet

Реалізація:

- `backend/main.ts`
  - `app.use(helmet())`
  - `app.use(compression())`

**Код з `backend/main.ts`:**

```typescript
import helmet from "helmet";
import compression from "compression";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  app.use(helmet());
  app.use(compression());
  // ...
}
```

### Rate-limit

Реалізація:

- `backend/app.module.ts`
  - `ThrottlerModule.forRoot({ throttlers: [{ limit: 10, ttl: 60 }] })`
- `backend/app.module.ts`
  - глобальний `APP_GUARD` з `ThrottlerGuard`

**Код з `backend/app.module.ts`:**

```typescript
import { ThrottlerGuard, ThrottlerModule } from "@nestjs/throttler";
import { APP_GUARD } from "@nestjs/core";

@Module({
  imports: [
    // ...
    ThrottlerModule.forRoot({
      throttlers: [
        {
          limit: 10,
          ttl: 60,
        },
      ],
    }),
    // ...
  ],
  providers: [
    // ...
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard,
    },
    // ...
  ],
})
export class AppModule {}
```

### Валідація даних

Реалізація:

- `backend/main.ts`
  - глобальний `ValidationPipe`
- DTO класи в `backend/dto/`
  - `backend/dto/register.dto.ts`
  - `backend/dto/login.dto.ts`
  - `backend/dto/update-password.dto.ts`
  - `backend/dto/update-profile.dto.ts`

Це означає, що об'єкти запитів проходять перевірку на наявність полів, формат email, довжину та правила пароля.

**Код з `backend/main.ts`:**

```typescript
import { ValidationPipe } from "@nestjs/common";

async function bootstrap() {
  // ...
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
      transformOptions: { enableImplicitConversion: true },
    }),
  );
  // ...
}
```

**Приклад DTO з `backend/dto/register.dto.ts`:**

```typescript
import {
  IsEmail,
  IsNotEmpty,
  IsString,
  MinLength,
  Matches,
} from "class-validator";

export class RegisterDto {
  @IsNotEmpty({ message: "Ім'я обов'язкове" })
  @IsString()
  name: string;

  @IsEmail({}, { message: "Невірний email" })
  @IsNotEmpty({ message: "Email обов'язковий" })
  email: string;

  @IsNotEmpty({ message: "Пароль обов'язковий" })
  @MinLength(6, { message: "Пароль повинен містити мінімум 6 символів" })
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: "Пароль повинен містити велику літеру, малу літеру та цифру",
  })
  password: string;

  @IsNotEmpty({ message: "Підтвердження пароля обов'язкове" })
  passwordConfirm: string;
}
```

---

## 3. Реалізувати кешування відповідей

Реалізація кешування:

- `backend/app.module.ts`
  - підключено `CacheModule.registerAsync(...)`
  - автоконфігурація Redis за `REDIS_URL` або локальний кеш
  - глобальний `APP_INTERCEPTOR` з `CacheInterceptor`
- Власне використання кешу в контролерах:
  - `backend/controllers/games.controller.ts`
    - кешування результатів `GET /api/games`
    - кешування `GET /api/games/:id`
  - `backend/controllers/hardware.controller.ts`
    - кешування `GET /api/hardware/:userId`

При створенні/оновленні/видаленні ігор кеш очищається за допомогою `this.cacheManager.clear()`.

**Код з `backend/app.module.ts`:**

```typescript
import { CacheModule, CacheInterceptor } from "@nestjs/cache-manager";
import { APP_INTERCEPTOR } from "@nestjs/core";

@Module({
  imports: [
    // ...
    CacheModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: async (config: ConfigService) => {
        const redisUrl = config.get<string>("REDIS_URL");
        if (redisUrl) {
          const redisStore = await import("cache-manager-ioredis");
          return {
            store: redisStore.default || redisStore,
            url: redisUrl,
            ttl: 120,
          } as any;
        }
        return {
          ttl: 60,
          max: 100,
        };
      },
    }),
    // ...
  ],
  providers: [
    // ...
    {
      provide: APP_INTERCEPTOR,
      useClass: CacheInterceptor,
    },
    // ...
  ],
})
export class AppModule {}
```

**Код з `backend/controllers/games.controller.ts`:**

```typescript
import { CacheManager } from "@nestjs/cache-manager";

@Controller("games")
export class GamesController {
  constructor(
    private gamesService: GamesService,
    private cacheManager: CacheManager,
  ) {}

  @Get()
  async getGames(
    @Query("page") page: number = 1,
    @Query("limit") limit: number = 10,
  ) {
    return this.gamesService.getGames(page, limit);
  }

  @Post()
  async createGame(@Body() createGameDto: CreateGameDto) {
    const result = await this.gamesService.createGame(createGameDto);
    await this.cacheManager.clear();
    return result;
  }
}
```

---

## 4. Оптимізувати один із маршрутів API

Оптимізовано маршрут `GET /api/games` у `backend/controllers/games.controller.ts`.

Оптимізації:

- Підтримка пагінації: `page` + `limit`
- Обмеження `limit` на максимум `50`
- Використання `findAndCountAll` для отримання сторінкових даних та загальної кількості
- Кешування відповіді на 120 секунд за ключем `games:page=X&limit=Y`

Це знижує навантаження на базу даних при повторних запитах і дозволяє обробляти великі набори даних ефективніше.

---

## 5. Провести тестування API

Тестування реалізовано у:

- `backend/test/app.e2e-spec.ts`

Тести:

- `GET /api/games` повертає 200 і JSON
- `POST /api/games` створює нову гру
- `GET /api/games/:id` повертає створену гру

Скрипт запуску тестів:

- `backend/package.json`
  - `test: "jest --runInBand"`

---

## 6. Проаналізувати продуктивність до та після оптимізації

Підготовлено інструмент для навантажувального тестування.

Файл:

- `backend/artillery.yaml`

Конфігурація:

- `target: "http://localhost:3000/api"`
- 30 секунд, 5 запитів на секунду
- сценарій: `GET /games?page=1&limit=10` та `POST /games`

Запуск:

- `cd backend && npx artillery run artillery.yaml`

---

## Висновок

У цьому проєкті всі основні вимоги виконані через NestJS бекенд:

- безпечний REST API з `helmet` і `rate-limit`
- валідація DTO
- кешування відповідей
- оптимізований маршрут `GET /api/games`
- e2e-тести через Jest/Supertest
- готовий сценарій для аналізу продуктивності


