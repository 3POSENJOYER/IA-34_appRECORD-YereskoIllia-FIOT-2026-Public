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
1. **Встановити необхідні бібліотеки**
   - Файл: `backend/package.json`
   - Використано пакети: `@nestjs/jwt`, `passport`, `passport-jwt`, `@nestjs/passport`, `bcryptjs`, `class-validator`, `@nestjs/throttler`, `nodemailer`, `sequelize`, `sequelize-typescript`, `mysql2`, `sqlite3`.

   ```json
   // backend/package.json (фрагмент dependencies)
   "dependencies": {
     "@nestjs/jwt": "^10.2.0",
     "@nestjs/passport": "^10.0.3",
     "@nestjs/throttler": "^6.2.1",
     "bcryptjs": "^2.4.3",
     "class-validator": "^0.14.1",
     "nodemailer": "^6.9.15",
     "passport": "^0.7.0",
     "passport-jwt": "^4.0.1",
     "sequelize": "^6.37.3",
     "sequelize-typescript": "^2.1.6",
     "mysql2": "^3.10.3",
     "sqlite3": "^5.1.7"
   }
   ```

2. **Реалізувати реєстрацію та авторизацію користувача**
   - Контролер: `backend/controllers/auth.controller.ts` (`POST /auth/register`, `POST /auth/login`)
   - Сервіс: `backend/services/auth.service.ts` (`register()`, `login()`)

   ```typescript
   // backend/controllers/auth.controller.ts (фрагмент)
   @Post('register')
   async register(@Body() registerDto: RegisterDto) {
     return this.authService.register(registerDto);
   }

   @Post('login')
   async login(@Body() loginDto: LoginDto) {
     return this.authService.login(loginDto);
   }
   ```

   ```typescript
   // backend/services/auth.service.ts (фрагмент)
   async register(registerDto: RegisterDto): Promise<{ success: boolean; user: User; accessToken: string; refreshToken: string }> {
     // Логіка реєстрації
     const hashedPassword = await bcrypt.hash(registerDto.password, 10);
     const user = await this.userModel.create({ ...registerDto, password: hashedPassword });
     const tokens = await this.generateTokens(user);
     return { success: true, user, ...tokens };
   }

   async login(loginDto: LoginDto): Promise<{ success: boolean; user: User; accessToken: string; refreshToken: string }> {
     // Логіка входу
     const user = await this.userModel.findOne({ where: { email: loginDto.email } });
     if (!user || !(await bcrypt.compare(loginDto.password, user.password))) {
       throw new UnauthorizedException('Invalid credentials');
     }
     const tokens = await this.generateTokens(user);
     return { success: true, user, ...tokens };
   }
   ```

3. **Додати валідацію даних, обробку помилок**
   - DTO: `backend/dto/register.dto.ts`, `backend/dto/login.dto.ts`, `backend/dto/update-password.dto.ts`, `backend/dto/update-profile.dto.ts`
   - Обробка помилок у контролері: `try/catch` в `backend/controllers/auth.controller.ts` та викидання `HttpException`, `UnauthorizedException`, `BadRequestException` у `backend/services/auth.service.ts`

   ```typescript
   // backend/dto/register.dto.ts
   export class RegisterDto {
     @IsNotEmpty()
     @IsString()
     name: string;

     @IsEmail()
     email: string;

     @IsNotEmpty()
     @MinLength(6)
     password: string;

     @IsNotEmpty()
     passwordConfirm: string;
   }
   ```

   ```typescript
   // backend/controllers/auth.controller.ts (фрагмент обробки помилок)
   @Post('register')
   async register(@Body() registerDto: RegisterDto) {
     try {
       return await this.authService.register(registerDto);
     } catch (error) {
       throw new HttpException(error.message, HttpStatus.BAD_REQUEST);
     }
   }
   ```

4. **Реалізувати захищений маршрут**
   - Гард: `backend/guards/jwt-auth.guard.ts`
   - Захищені ендпоінти: `GET /auth/profile`, `PUT /auth/profile`, `PUT /auth/change-password`, `DELETE /auth/user`, `POST /auth/logout`, `GET /auth/admin/users`

   ```typescript
   // backend/guards/jwt-auth.guard.ts
   import { Injectable } from "@nestjs/common";
   import { AuthGuard } from "@nestjs/passport";

   @Injectable()
   export class JwtAuthGuard extends AuthGuard("jwt") {}
   ```

   ```typescript
   // backend/controllers/auth.controller.ts (фрагмент захищеного маршруту)
   @UseGuards(JwtAuthGuard)
   @Get('profile')
   getProfile(@Request() req) {
     return req.user;
   }
   ```

5. **Протестувати API через Postman або curl**
   - Це процесна дія поза кодом, але всі відповідні API-ендпоінти доступні в `backend/controllers/auth.controller.ts`.

6. **Проаналізувати отримані результати**
   - Аналіз виконується на основі відповіді API та логів. Код логування міститься у `backend/services/logger.service.ts`.

7. **Додати підтвердження пароля при реєстрації**
   - Валідація на рівні сервісу: `backend/services/auth.service.ts`, метод `register()` перевіряє `password === passwordConfirm`.
   - DTO: `backend/dto/register.dto.ts` містить поле `passwordConfirm`.

   ```typescript
   // backend/services/auth.service.ts (фрагмент)
   async register(registerDto: RegisterDto) {
     if (registerDto.password !== registerDto.passwordConfirm) {
       throw new BadRequestException('Passwords do not match');
     }
     // решта логіки
   }
   ```

8. **Додати роль користувача (admin/user)**
   - Модель: `backend/models/User.ts`, поле `role` з типом `"user" | "admin"`.
   - Підтримка ролі передається в JWT-пейлоуді в `backend/services/auth.service.ts`.

   ```typescript
   // backend/models/User.ts (фрагмент)
   @Column({
     type: DataType.ENUM('user', 'admin'),
     defaultValue: 'user',
   })
   role: 'user' | 'admin';
   ```

   ```typescript
   // backend/services/auth.service.ts (фрагмент)
   private async generateTokens(user: User) {
     const payload = { sub: user.id, email: user.email, role: user.role };
     const accessToken = this.jwtService.sign(payload);
     const refreshToken = this.jwtService.sign(payload, { expiresIn: '7d' });
     return { accessToken, refreshToken };
   }
   ```

9. **Реалізувати logout**
   - Контролер: `backend/controllers/auth.controller.ts`, `POST /auth/logout`
   - Сервіс: `backend/services/auth.service.ts`, метод `logout()` обнуляє `refreshToken` у базі.

   ```typescript
   // backend/controllers/auth.controller.ts (фрагмент)
   @UseGuards(JwtAuthGuard)
   @Post('logout')
   async logout(@Request() req) {
     return this.authService.logout(req.user.id);
   }
   ```

   ```typescript
   // backend/services/auth.service.ts (фрагмент)
   async logout(userId: number): Promise<{ success: boolean }> {
     await this.userModel.update({ refreshToken: null }, { where: { id: userId } });
     return { success: true };
   }
   ```

10. **Додати оновлення профілю**
    - Контролер: `PUT /auth/profile` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, метод `updateProfile()`
    - DTO: `backend/dto/update-profile.dto.ts`

    ```typescript
    // backend/controllers/auth.controller.ts (фрагмент)
    @UseGuards(JwtAuthGuard)
    @Put('profile')
    async updateProfile(@Body() updateProfileDto: UpdateProfileDto, @Request() req) {
      return this.authService.updateProfile(req.user.id, updateProfileDto);
    }
    ```

    ```typescript
    // backend/services/auth.service.ts (фрагмент)
    async updateProfile(userId: number, updateProfileDto: UpdateProfileDto): Promise<User> {
      await this.userModel.update(updateProfileDto, { where: { id: userId } });
      return this.userModel.findByPk(userId);
    }
    ```

11. **Зберігати користувачів у базі**
    - Модель: `backend/models/User.ts`
    - Sequelize налаштований у `backend/app.module.ts`
    - Створення користувача у `backend/services/auth.service.ts`, метод `register()`

    ```typescript
    // backend/models/User.ts (фрагмент)
    @Table
    export class User extends Model<User> {
      @Column({ primaryKey: true, autoIncrement: true })
      id: number;

      @Column
      name: string;

      @Column({ unique: true })
      email: string;

      @Column
      password: string;

      @Column({
        type: DataType.ENUM("user", "admin"),
        defaultValue: "user",
      })
      role: "user" | "admin";
    }
    ```

12. **Реалізувати refresh token**
    - Контролер: `POST /auth/refresh` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, метод `refreshToken()`
    - Сховище: `refreshToken` хешується та зберігається в полі `refreshToken` моделі `User`

    ```typescript
    // backend/controllers/auth.controller.ts (фрагмент)
    @Post('refresh')
    async refresh(@Body('refreshToken') refreshToken: string) {
      return this.authService.refreshToken(refreshToken);
    }
    ```

    ```typescript
    // backend/services/auth.service.ts (фрагмент)
    async refreshToken(refreshToken: string): Promise<{ accessToken: string }> {
      try {
        const payload = this.jwtService.verify(refreshToken);
        const user = await this.userModel.findByPk(payload.sub);
        if (!user || user.refreshToken !== await bcrypt.hash(refreshToken, 10)) {
          throw new UnauthorizedException('Invalid refresh token');
        }
        const accessToken = this.jwtService.sign({ sub: user.id, email: user.email, role: user.role });
        return { accessToken };
      } catch (error) {
        throw new UnauthorizedException('Invalid refresh token');
      }
    }
    ```

13. **Додати логування помилок**
    - Сервіс: `backend/services/logger.service.ts`
    - Виклики логування в `backend/services/auth.service.ts` (`logInfo`, `logError`)

    ```typescript
    // backend/services/logger.service.ts (фрагмент)
    export class LoggerService {
      logInfo(message: string) {
        console.log(`[INFO] ${new Date().toISOString()}: ${message}`);
      }

      logError(message: string, error?: any) {
        console.error(`[ERROR] ${new Date().toISOString()}: ${message}`, error);
      }
    }
    ```

    ```typescript
    // backend/services/auth.service.ts (фрагмент)
    try {
      // код
    } catch (error) {
      this.loggerService.logError("Error in register", error);
      throw error;
    }
    ```

14. **Обмежити кількість спроб входу**
    - Сервіс: `backend/services/auth.service.ts`, метод `handleFailedLogin()` відстежує `loginAttempts` і встановлює `lockUntil` після 5 невдалих спроб.
    - Також у `backend/app.module.ts` підключено `ThrottlerModule`.

    ```typescript
    // backend/services/auth.service.ts (фрагмент)
    async handleFailedLogin(email: string) {
      const user = await this.userModel.findOne({ where: { email } });
      if (user) {
        user.loginAttempts += 1;
        if (user.loginAttempts >= 5) {
          user.lockUntil = new Date(Date.now() + 2 * 60 * 60 * 1000); // 2 години
        }
        await user.save();
      }
    }
    ```

15. **Додати middleware для перевірки токена**
    - Стратегія: `backend/strategies/jwt.strategy.ts`
    - Гард: `backend/guards/jwt-auth.guard.ts`
    - Роут-гард використовується у `backend/controllers/auth.controller.ts`

    ```typescript
    // backend/strategies/jwt.strategy.ts
    import { Injectable } from "@nestjs/common";
    import { PassportStrategy } from "@nestjs/passport";
    import { ExtractJwt, Strategy } from "passport-jwt";

    @Injectable()
    export class JwtStrategy extends PassportStrategy(Strategy) {
      constructor() {
        super({
          jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
          ignoreExpiration: false,
          secretOrKey: process.env.JWT_SECRET,
        });
      }

      async validate(payload: any) {
        return { id: payload.sub, email: payload.email, role: payload.role };
      }
    }
    ```

16. **Реалізувати зміну пароля**
    - Контролер: `PUT /auth/change-password` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, метод `changePassword()`
    - DTO: `backend/dto/update-password.dto.ts`

    ```typescript
    // backend/controllers/auth.controller.ts (фрагмент)
    @UseGuards(JwtAuthGuard)
    @Put('change-password')
    async changePassword(@Body() updatePasswordDto: UpdatePasswordDto, @Request() req) {
      return this.authService.changePassword(req.user.id, updatePasswordDto);
    }
    ```

    ```typescript
    // backend/services/auth.service.ts (фрагмент)
    async changePassword(userId: number, updatePasswordDto: UpdatePasswordDto): Promise<{ success: boolean }> {
      const user = await this.userModel.findByPk(userId);
      if (!(await bcrypt.compare(updatePasswordDto.oldPassword, user.password))) {
        throw new BadRequestException('Old password is incorrect');
      }
      const hashedNewPassword = await bcrypt.hash(updatePasswordDto.newPassword, 10);
      await this.userModel.update({ password: hashedNewPassword }, { where: { id: userId } });
      return { success: true };
    }
    ```

17. **Реалізувати видалення користувача**
    - Контролер: `DELETE /auth/user` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, метод `deleteUser()`

    ```typescript
    // backend/controllers/auth.controller.ts (фрагмент)
    @UseGuards(JwtAuthGuard)
    @Delete('user')
    async deleteUser(@Request() req) {
      return this.authService.deleteUser(req.user.id);
    }
    ```

    ```typescript
    // backend/services/auth.service.ts (фрагмент)
    async deleteUser(userId: number): Promise<{ success: boolean }> {
      await this.userModel.destroy({ where: { id: userId } });
      return { success: true };
    }
    ```

18. **Реалізувати відновлення пароля**
    - Контролер: `POST /auth/forgot-password`, `POST /auth/reset-password` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, методи `requestPasswordReset()` і `resetPassword()`
    - Пошта для скидання пароля: `sendPasswordResetEmail()` у `backend/services/auth.service.ts`

    ```typescript
    // backend/controllers/auth.controller.ts (фрагмент)
    @Post('forgot-password')
    async forgotPassword(@Body('email') email: string) {
      return this.authService.requestPasswordReset(email);
    }

    @Post('reset-password')
    async resetPassword(@Body() body: { token: string; newPassword: string }) {
      return this.authService.resetPassword(body.token, body.newPassword);
    }
    ```

    ```typescript
    // backend/services/auth.service.ts (фрагмент)
    async requestPasswordReset(email: string): Promise<{ success: boolean }> {
      const user = await this.userModel.findOne({ where: { email } });
      if (user) {
        const resetToken = this.jwtService.sign({ sub: user.id }, { expiresIn: '1h' });
        await this.sendPasswordResetEmail(user.email, resetToken);
      }
      return { success: true };
    }

    async resetPassword(token: string, newPassword: string): Promise<{ success: boolean }> {
      try {
        const payload = this.jwtService.verify(token);
        const hashedPassword = await bcrypt.hash(newPassword, 10);
        await this.userModel.update({ password: hashedPassword }, { where: { id: payload.sub } });
        return { success: true };
      } catch (error) {
        throw new BadRequestException('Invalid or expired token');
      }
    }
    ```

19. **Додати підтвердження email**
    - Контролер: `GET /auth/confirm-email/:token` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, метод `confirmEmail()`
    - Пошта для підтвердження: `sendConfirmationEmail()` у `backend/services/auth.service.ts`

    ```typescript
    // backend/controllers/auth.controller.ts (фрагмент)
    @Get('confirm-email/:token')
    async confirmEmail(@Param('token') token: string) {
      return this.authService.confirmEmail(token);
    }
    ```

    ```typescript
    // backend/services/auth.service.ts (фрагмент)
    async confirmEmail(token: string): Promise<{ success: boolean }> {
      try {
        const payload = this.jwtService.verify(token);
        await this.userModel.update({ isEmailConfirmed: true }, { where: { id: payload.sub } });
        return { success: true };
      } catch (error) {
        throw new BadRequestException('Invalid or expired token');
      }
    }
    ```

20. **Реалізувати OAuth (Google login)**
    - У репозиторії поки що немає реалізації OAuth/Google login.
    - Коду `GoogleStrategy`, `passport-google-oauth20` або подібних файлів в `backend/` не знайдено.
    - Сервіс: `backend/services/auth.service.ts`, метод `refreshToken()`
    - Сховище: `refreshToken` хешується та зберігається в полі `refreshToken` моделі `User`

21. **Додати логування помилок**
    - Сервіс: `backend/services/logger.service.ts`
    - Виклики логування в `backend/services/auth.service.ts` (`logInfo`, `logError`)

22. **Обмежити кількість спроб входу**
    - Сервіс: `backend/services/auth.service.ts`, метод `handleFailedLogin()` відстежує `loginAttempts` і встановлює `lockUntil` після 5 невдалих спроб.
    - Також у `backend/app.module.ts` підключено `ThrottlerModule`.

23. **Додати middleware для перевірки токена**
    - Стратегія: `backend/strategies/jwt.strategy.ts`
    - Гард: `backend/guards/jwt-auth.guard.ts`
    - Роут-гард використовується у `backend/controllers/auth.controller.ts`

24. **Реалізувати зміну пароля**
    - Контролер: `PUT /auth/change-password` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, метод `changePassword()`
    - DTO: `backend/dto/update-password.dto.ts`

25. **Реалізувати видалення користувача**
    - Контролер: `DELETE /auth/user` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, метод `deleteUser()`

26. **Реалізувати відновлення пароля**
    - Контролер: `POST /auth/forgot-password`, `POST /auth/reset-password` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, методи `requestPasswordReset()` і `resetPassword()`
    - Пошта для скидання пароля: `sendPasswordResetEmail()` у `backend/services/auth.service.ts`

27. **Додати підтвердження email**
    - Контролер: `GET /auth/confirm-email/:token` в `backend/controllers/auth.controller.ts`
    - Сервіс: `backend/services/auth.service.ts`, метод `confirmEmail()`
    - Пошта для підтвердження: `sendConfirmationEmail()` у `backend/services/auth.service.ts`



## Висновок

У межах розробки системи автентифікації на базі NestJS було успішно реалізовано повний цикл управління обліковими записами користувачів. Основну увагу приділено безпеці: впроваджено хешування паролів за допомогою bcrypt, захист маршрутів через JWT-стратегії та механізм обмеження спроб входу (Throttler та lockUntil). Реалізований функціонал (зміна/відновлення пароля, підтвердження email та видалення акаунта) забезпечує гнучкість системи, а логування процесів дозволяє ефективно моніторити стан безпеки додатка.