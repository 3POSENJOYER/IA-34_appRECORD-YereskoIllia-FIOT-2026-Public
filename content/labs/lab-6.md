## Тема, Мета, Місце розташування

**Тема:** Розробка адаптивного веб-застосунку «GameStack» для агрегації ігрових бібліотек та аналізу сумісності з апаратним забезпеченням.

**Мета:** На основі досвіду розроблення адаптивних інтерфейсів створити платформу для геймерів, що дозволяє переглядати ігри з різних сервісів (Steam, Epic, GOG) та отримувати прогнози продуктивності (FPS) залежно від заліза користувача.

**Місце розташування:**

- **GitHub:** [https://github.com/3POSENJOYER/IA-34_appRECORD-YereskoIllia-FIOT-2026-Public](https://github.com/3POSENJOYER/IA-34_appRECORD-YereskoIllia-FIOT-2026-Public)
- **Live demo:** [https://3posenjoyer.github.io/IA-34_appRECORD-YereskoIllia-FIOT-2026-Public/](https://3posenjoyer.github.io/IA-34_appRECORD-YereskoIllia-FIOT-2026-Public/)

---

## Опис предметного середовища

Предметне середовище проекту «GameStack» охоплює сферу цифрової дистрибуції відеоігор та технічного аналізу апаратних ресурсів персональних комп'ютерів. Основним проблемним аспектом у цій галузі є високий рівень фрагментації контенту, оскільки провідні цифрові магазини, такі як Steam, Epic Games Store та GOG, використовують закриті екосистеми та власні клієнтські додатки для керування контентом. Це створює значні незручності для кінцевого користувача, який змушений оперувати декількома бібліотеками одночасно без можливості швидкої навігації або централізованого пошуку. Крім того, стрімкий розвиток ігрової індустрії та постійне зростання графічних вимог до програмних продуктів роблять критично важливим процес попереднього оцінювання сумісності гри з наявним відеоадаптером та процесором користувача. Даний проект спрямований на розв'язання цих проблем шляхом створення інтелектуальної платформи, яка виступає єдиним інтерфейсом для агрегації даних та автоматизованого розрахунку технічних метрик продуктивності. Впровадження такої системи дозволяє геймерам уникати помилок при виборі контенту, забезпечуючи прозорий механізм прогнозування частоти кадрів та загальної стабільності ігрового процесу на основі реальних характеристик заліза, що робить взаємодію з ігровими бібліотеками більш ефективною та передбачуваною.

---

## Структура веб-застосунку

Архітектура проекту «GameStack» побудована за сучасним компонентним принципом, що забезпечує чіткий розподіл відповідальності між модулями та високу швидкість розробки. Коренева директорія містить конфігураційні файли для збірки проекту через Vite та налаштування пакетного менеджера pnpm, тоді як основний код зосереджений у папці src. Внутрішня організація передбачає винесення ізольованих UI-елементів, таких як адаптивна шапка та інтерактивні картки ігор, у директорію компонентів, що дозволяє перевикористовувати логіку відображення. Глобальна стилізація реалізована за допомогою модульного SCSS, де окремі файли відповідають за змінні кольорів, брейкпоінти для адаптивності та базові правила верстки. Типізація даних забезпечується окремим модулем з TypeScript-інтерфейсами, які жорстко описують структуру об’єктів ігор та параметрів обладнання, гарантуючи стабільність роботи всього застосунку. Додатково структура включає директорію для статичних ресурсів та спеціалізовану папку налаштувань GitHub Actions, яка автоматизує процеси тестування та розгортання готового продукту на серверах GitHub Pages.

---

## Сценарій взаємодії (бізнес-логіка)

Процес взаємодії з платформою розпочинається з етапу авторизації та синхронізації, під час якого система через зовнішні API-інтерфейси ігрових сервісів отримує дані про бібліотеку користувача та формує локальний кеш із метаданими ігор. Одночасно з цим застосунок ініціює профільування апаратного забезпечення, зчитуючи характеристики графічного та центрального процесорів для створення індивідуального профілю потужності. Ключова логіка аналізу базується на динамічному зіставленні отриманого профілю із системними вимогами кожної гри, результатом якого є обчислення очікуваної частоти кадрів (FPS) та рівня сумісності.

## Вимоги

### Функціональні вимоги:

- **FR-1:** Агрегація ігрових даних з декількох джерел (Steam, Epic Games Store, GOG).
- **FR-2:** Аналіз сумісності вимог гри із залізом користувача (CPU, GPU, RAM).
- **FR-3:** Прогнозування FPS на основі алгоритму порівняння потужності.
- **FR-4:** Система фільтрації за назвою та платформою.
- **FR-5:** Візуалізація метрик через прогрес-бари зі зміною кольору.
- **FR-6:** Деталізація заліза поточного користувача на головній панелі.

### Нефункціональні вимоги:

- **NFR-1:** Дизайн у стилі Gaming UI (темна тема, неонові акценти).
- **NFR-2:** Адаптивність (Desktop 1200px+, Tablet 768px+, Mobile &lt; 768px).
- **NFR-3:** Продуктивність: час ініціалізації додатку до 1.0 сек.
- **NFR-4:** Плавні CSS-анімації (transition, hover effects).
- **NFR-5:** Масштабованість для додавання нових платформ (Ubisoft, Battle.net).
- **NFR-6:** Доступність: використання семантичних HTML-тегів (`<article>`, `<nav>`, `<header>`) та відносних одиниць (`rem`, `em`, `%`).

## Стек технологій

- **Vue 3 (Composition API):** Основний фреймворк для побудови реактивного та модульного інтерфейсу користувача.
- **TypeScript:** Мова програмування, що забезпечує строгу типізацію даних, покращує стабільність коду та полегшує розробку складних структур даних.
- **Vite:** Сучасний інструмент збірки та сервер розробки, що забезпечує миттєве оновлення модулів (HMR) та високу швидкість компіляції.
- **SCSS (Sass):** Препроцесор для стилізації, який використовується для створення гнучкої темної теми, керування змінними та побудови адаптивної верстки.
- **pnpm:** Ефективний менеджер пакетів, що оптимізує використання дискового простору та пришвидшує встановлення залежностей.
- **GitHub Actions:** Інструмент для автоматизації CI/CD процесів, що забезпечує автоматичну збірку та деплой проєкту.


# Звіт виконання завдань REST API з MySQL та Swagger

## Завдання 1. Створити REST API на Node.js + Express

Проєкт реалізовано у папці `backend` з використанням NestJS, яка працює на базі `Express`.

- `backend/main.ts` — старт сервера та налаштування глобальних middleware.
- `backend/app.module.ts` — модуль, який збирає контролери, службу, та підключення до бази даних.

Приклад з `backend/main.ts`:

```ts
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

> NestJS автоматично використовує Express як платформу через `@nestjs/platform-express`.

## Завдання 2. Підключити MySQL. Реалізувати CRUD-операції

Для підключення MySQL використано `@nestjs/sequelize` та `sequelize-typescript`.

- `backend/app.module.ts` — конфігурація Sequelize з MySQL та SQLite як fallback.
- `docker-compose.yml` — підключено сервіс `mysql`.
- `backend/models/Game.ts` — модель `Game`.
- `backend/controllers/games.controller.ts` — реалізовані CRUD-ендпоінти для ігор.

Приклад підключення MySQL у `backend/app.module.ts`:

```ts
SequelizeModule.forRootAsync({
  imports: [ConfigModule],
  inject: [ConfigService],
  useFactory: (config: ConfigService): SequelizeModuleOptions => {
    const dbHost = config.get<string>("DB_HOST");

    if (dbHost) {
      return {
        dialect: "mysql",
        host: dbHost,
        port: Number(config.get<number>("DB_PORT")) || 3306,
        username: config.get<string>("DB_USER"),
        password: config.get<string>("DB_PASSWORD"),
        database: config.get<string>("DB_NAME"),
        models: [User as any, Post as any, Game as any, HardwareProfile as any],
        autoLoadModels: true,
        synchronize: true,
        logging: false,
      };
    }

    return {
      dialect: "sqlite",
      storage: "./database.sqlite",
      models: [User as any, Post as any, Game as any, HardwareProfile as any],
      autoLoadModels: true,
      synchronize: true,
      logging: false,
    };
  },
}),
```

Файл `docker-compose.yml`:

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: web_backend_lab
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  node:
    build: ./backend
    ports:
      - "3000:3000"
    depends_on:
      - mysql
    environment:
      - DB_HOST=mysql
      - DB_USER=root
      - DB_PASSWORD=password
      - DB_NAME=web_backend_lab

volumes:
  mysql_data:
```

CRUD-ендпоінти для `Game` у `backend/controllers/games.controller.ts`:

```ts
@Get()
async findAll(...) { ... }

@Get(":id")
async findOne(@Param("id") id: string) { ... }

@Post()
async create(@Body() gameData: CreateGameDto): Promise<Game> { ... }

@Put(":id")
async update(@Param("id") id: string, @Body() gameData: UpdateGameDto): Promise<Game> { ... }

@Delete(":id")
async remove(@Param("id") id: string): Promise<void> { ... }
```

## Завдання 3. Інтегрувати Swagger UI у проєкт

Swagger UI інтегровано у `backend/main.ts` за допомогою `@nestjs/swagger`.

- Пакунки: `@nestjs/swagger`, `swagger-ui-express`.
- URL документації: `http://localhost:3000/api/docs`.

Фрагмент налаштування Swagger:

```ts
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
```

## Завдання 4. Задокументувати всі endpoint-и

Документація додається до контролерів через Swagger-декоратори.

Приклади у контролерах:

- `backend/controllers/auth.controller.ts`
- `backend/controllers/games.controller.ts`
- `backend/controllers/hardware.controller.ts`

Фрагмент документації у `auth.controller.ts`:

```ts
@ApiTags("auth")
@Post("register")
@HttpCode(201)
@ApiOperation({ summary: "Register a new user" })
@ApiBody({ type: RegisterDto })
@ApiResponse({ status: 201, description: "User successfully registered." })
@ApiResponse({ status: 400, description: "Validation error or registration failure." })
async register(@Body() registerDto: RegisterDto) { ... }
```

Фрагмент документації у `games.controller.ts`:

```ts
@ApiTags("games")
@Get(":id")
@ApiOperation({ summary: "Get game by ID" })
@ApiParam({ name: "id", description: "Game ID" })
@ApiResponse({ status: 200, description: "Game found." })
@ApiResponse({ status: 404, description: "Game not found." })
async findOne(@Param("id") id: string): Promise<Game | null> { ... }
```

## Завдання 5. Виконати тестування API через Swagger UI

API можна тестувати вручну через Swagger UI:

- Відкрити: `http://localhost:3000/api/docs`
- Виконати POST `/api/auth/register`
- Виконати POST `/api/auth/login`
- Виконати CRUD на `/api/games`
- Виконати POST `/api/hardware`

Це частина кінцевої демонстрації, де всі endpoint-и доступні для тесту в UI.

## Завдання 6. Продемонструвати підсумковий проєкт: REST API з MySQL та Swagger документацією

Підсумковий проєкт містить:

- `backend/main.ts` — запуск API, глобальні middleware, Swagger UI
- `backend/app.module.ts` — підключення MySQL/Sequelize, моделі, контролери
- `backend/controllers` — реалізація REST API для `auth`, `games`, `hardware`, `predict`
- `backend/models` — Sequelize-моделі `User`, `Game`, `HardwareProfile`, `Post`
- `docker-compose.yml` — підняття MySQL та Node-сервера
- `backend/.env.example` — приклад конфігурації БД та JWT




