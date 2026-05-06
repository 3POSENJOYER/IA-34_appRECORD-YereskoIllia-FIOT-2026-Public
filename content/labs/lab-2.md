# Лабораторна робота №2

## Тема, мета, місце розташування

**Тема:** Розробка адаптивного веб-застосунку «GameStack» для агрегації ігрових бібліотек та аналізу сумісності з апаратним забезпеченням.

**Мета:** На основі досвіду розроблення адаптивних інтерфейсів створити платформу для геймерів, що дозволяє переглядати ігри з різних сервісів (Steam, Epic, GOG) та отримувати прогнози продуктивності (FPS) залежно від заліза користувача.

**Місце розташування:**

- **GitHub:** [https://github.com/3POSENJOYER/IA-34_appRECORD-YereskoIllia-FIOT-2026-Public](https://github.com/3POSENJOYER/IA-34_appRECORD-YereskoIllia-FIOT-2026-Public)
- **Live demo:** [https://3posenjoyer.github.io/IA-34_appRECORD-YereskoIllia-FIOT-2026-Public/](https://3posenjoyer.github.io/IA-34_appRECORD-YereskoIllia-FIOT-2026-Public/)

---

## Опис предметного середовища

Предметне середовище проєкту «GameStack» охоплює сферу цифрової дистрибуції відеоігор та технічного аналізу апаратних ресурсів персональних комп'ютерів. Основним проблемним аспектом у цій галузі є високий рівень фрагментації контенту, оскільки провідні цифрові магазини, такі як Steam, Epic Games Store та GOG, використовують закриті екосистеми та власні клієнтські додатки для керування контентом.

Це створює значні незручності для кінцевого користувача, який змушений оперувати декількома бібліотеками одночасно без можливості швидкої навігації або централізованого пошуку.

Крім того, стрімкий розвиток ігрової індустрії та постійне зростання графічних вимог до програмних продуктів роблять критично важливим процес попереднього оцінювання сумісності гри з наявним відеоадаптером та процесором користувача.

Даний проєкт спрямований на розв'язання цих проблем шляхом створення інтелектуальної платформи, яка виступає єдиним інтерфейсом для агрегації даних та автоматизованого розрахунку технічних метрик продуктивності.

Впровадження такої системи дозволяє геймерам уникати помилок при виборі контенту, забезпечуючи прозорий механізм прогнозування частоти кадрів та загальної стабільності ігрового процесу на основі реальних характеристик заліза.

---

## Вимоги

### Функціональні вимоги

- **FR-1:** Агрегація ігрових даних з декількох джерел (Steam, Epic Games Store, GOG).
- **FR-2:** Аналіз сумісності вимог гри із залізом користувача (CPU, GPU, RAM).
- **FR-3:** Прогнозування FPS на основі алгоритму порівняння потужності.
- **FR-4:** Система фільтрації за назвою та платформою.
- **FR-5:** Візуалізація метрик через прогрес-бари зі зміною кольору.
- **FR-6:** Деталізація заліза поточного користувача на головній панелі.

### Нефункціональні вимоги

- **NFR-1:** Дизайн у стилі Gaming UI (темна тема, неонові акценти).
- **NFR-2:** Адаптивність (Desktop 1200px+, Tablet 768px+, Mobile < 768px).
- **NFR-3:** Продуктивність: час ініціалізації додатку до 1.0 сек.
- **NFR-4:** Плавні CSS-анімації (transition, hover effects).
- **NFR-5:** Масштабованість для додавання нових платформ (Ubisoft, Battle.net).
- **NFR-6:** Доступність: використання семантичних HTML-тегів (`<article>`, `<nav>`, `<header>`) та відносних одиниць (`rem`, `em`, `%`).

---

## Стек технологій

- **Vue 3 (Composition API):** основний фреймворк для побудови реактивного та модульного інтерфейсу користувача.
- **TypeScript:** мова програмування для суворої типізації та стабільності коду.
- **Vite:** сучасний інструмент збірки та сервер розробки для швидкої компіляції та HMR.
- **SCSS (Sass):** препроцесор для створення гнучкої темної теми й адаптивних стилів.
- **pnpm:** менеджер пакетів, що оптимізує використання дискового простору.
- **GitHub Actions:** інструмент для автоматизації CI/CD процесів.


## Виконання завдання

У межах лабораторної роботи було реалізовано backend-частину вебзастосунку **MacShnaknels** з підключенням до бази даних MySQL.

### 1. Створення бази даних

База даних створюється автоматично через Docker Compose. У файлі `docker-compose.yml` визначено MySQL-сервіс:

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: web_backend_lab  # Назва БД
      MYSQL_USER: user
      MYSQL_PASSWORD: pass
    ports:
      - "3306:3306"
```

При запуску `docker-compose up` створюється БД `web_backend_lab` з користувачем `user` та паролем `pass`.

### 2. Створення таблиць

Таблиці створюються автоматично через Sequelize ORM при старті додатку:

```ts
// backend/app.service.ts
async onModuleInit() {
  // Встановлення зв'язків між моделями
  (User as any).hasMany(Post, { foreignKey: "userId" });
  (Post as any).belongsTo(User, { foreignKey: "userId" });
  (User as any).hasOne(HardwareProfile, { foreignKey: "userId" });
  (HardwareProfile as any).belongsTo(User, { foreignKey: "userId" });

  await sequelize.sync({ force: true });  // Створює таблиці
  console.log("Tables created");

  await this.createInitialGames();  // Наповнення початковими даними
}
```

### 3. Виконання SQL-запитів (SELECT, INSERT, UPDATE, DELETE)

#### SELECT

```sql
SELECT id, name, email, role FROM users;
```

#### INSERT

```ts
// backend/services/auth.service.ts
async register(name: string, email: string, password: string): Promise<User> {
  const hashed = await bcrypt.hash(password, 10);
  return User.create({ name, email, password: hashed });  // INSERT
}
```

#### UPDATE

```ts
// backend/controllers/games.controller.ts
@Put(':id')
async update(@Param('id') id: string, @Body() updateGameDto: any): Promise<Game> {
  return this.gamesService.update(+id, updateGameDto);
}

// backend/services/games.service.ts
async update(id: number, updateGameDto: any): Promise<Game> {
  const game = await (Game as any).findByPk(id);
  return game.update(updateGameDto);  // UPDATE
}
```

#### DELETE

```ts
// backend/controllers/games.controller.ts
@Delete(':id')
async remove(@Param('id') id: string): Promise<void> {
  return this.gamesService.remove(+id);
}

// backend/services/games.service.ts
async remove(id: number): Promise<void> {
  const game = await (Game as any).findByPk(id);
  await game.destroy();  // DELETE
}
```

### 4. Підключення Node.js до MySQL через mysql2

Підключення налаштоване в `database.ts` через Sequelize, який використовує `mysql2` під капотом:

```ts
// backend/config/database.ts
import { Sequelize } from 'sequelize';

export const sequelize = new Sequelize(
  process.env.DB_NAME || 'web_backend_lab',
  process.env.DB_USER || 'root',
  process.env.DB_PASSWORD || 'password',
  {
    host: process.env.DB_HOST || 'localhost',
    dialect: 'mysql',  // Використовує mysql2 драйвер
  },
);
```

### 5. Виконання SQL-запитів з Node.js

Через Sequelize моделі виконуються SQL-запити. Наприклад, у `app.service.ts` створюються початкові ігри:

```ts
// backend/app.service.ts
async createInitialGames() {
  const gamesData = [
    { title: "Elden Ring", platform: "Steam" as const, ... },
    // ...
  ];

  for (const gameData of gamesData) {
    await (Game as any).create(gameData);  // INSERT запит
  }
}
```

### 6. Використання ORM Sequelize

Sequelize використовується як ORM для абстракції від сирих SQL-запитів. У проєкті:

- моделі визначені з декораторами `sequelize-typescript`;
- автоматичне створення/оновлення схеми через `sync()`;
- зв'язки між таблицями через `hasMany`, `belongsTo`, `hasOne`.

### 7. Створення моделей User та Post

Моделі створені з `sequelize-typescript` декораторами:

```ts
// backend/models/User.ts
@Table({ tableName: "users" })
export class User extends Model<UserAttributes, UserCreationAttributes> {
  @PrimaryKey
  @AutoIncrement
  @Column(DataType.INTEGER)
  public id!: number;

  @AllowNull(false)
  @Column(DataType.STRING)
  public name!: string;

  @AllowNull(false)
  @Unique
  @Column(DataType.STRING)
  public email!: string;

  @AllowNull(false)
  @Column(DataType.STRING)
  public password!: string;
}

// backend/models/Post.ts
@Table({ tableName: "posts" })
export class Post extends Model<PostAttributes, PostCreationAttributes> {
  @PrimaryKey
  @AutoIncrement
  @Column(DataType.INTEGER)
  public id!: number;

  @AllowNull(false)
  @Column(DataType.STRING)
  public title!: string;

  @AllowNull(false)
  @Column(DataType.TEXT)
  public content!: string;

  @ForeignKey(() => User)
  @AllowNull(false)
  @Column(DataType.INTEGER)
  public userId!: number;

  @BelongsTo(() => User)
  public user!: User;
}
```

### 8. Реалізація зв'язку One-to-Many

```ts
// backend/app.service.ts
(User as any).hasMany(Post, { foreignKey: "userId" });
(Post as any).belongsTo(User, { foreignKey: "userId" });
```

Це створює:

- `FOREIGN KEY` у таблиці `posts` на `users.id`.
- можливість отримання постів користувача: `user.getPosts()`.
- можливість отримання автора поста: `post.getUser()`.
- аналогічні зв'язки для `User ↔ HardwareProfile` (One-to-One).

---

## Висновки

Весь процес автоматизовано через Docker та Sequelize. При запуску `docker-compose up` створюється база даних, а при старті NestJS-додатку — таблиці та зв'язки. CRUD-операції реалізовані через контролери та сервіси з використанням Sequelize моделей замість сирих SQL-запитів.
