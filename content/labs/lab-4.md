# Звіт з виконання лабораторних завдань

## Загальний контекст

Проєкт реалізовано як NestJS бекенд у папці `backend`, але всі вимоги "REST API на Node.js" виконані у цьому серверному шарі. У звіті описано, де саме реалізовано кожне завдання.

---


## 1. Ініціалізація проєкту

Сервер створено у папці `backend` за допомогою NestJS поверх Express.

Файл: `backend/main.ts`

```typescript
import { NestFactory } from "@nestjs/core";
import { ValidationPipe } from "@nestjs/common";
import { SwaggerModule, DocumentBuilder } from "@nestjs/swagger";
import { NestExpressApplication } from "@nestjs/platform-express";
import helmet from "helmet";
import compression from "compression";
import morgan from "morgan";
import { AppModule } from "./app.module";
import { LoggerService } from "./services/logger.service";
import { AllExceptionsFilter } from "./filters/all-exceptions.filter";

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.enableCors();
  app.use(helmet());
  app.use(compression());
  app.use(morgan("combined"));
  app.setGlobalPrefix("api");

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
      transformOptions: { enableImplicitConversion: true },
    }),
  );

  app.useGlobalFilters(new AllExceptionsFilter(app.get(LoggerService)));

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

Файл: `backend/app.controller.ts`

```typescript
import { Controller, Get } from "@nestjs/common";

@Controller()
export class AppController {
  @Get()
  getHello() {
    return {
      message: "Hello World!",
      api: "/api",
      docs: "/api/docs",
    };
  }
}
```

Кореневий маршрут `/` повертає JSON відповідь.

## 2. Логування HTTP-запитів

HTTP-запити логуються в консоль через Morgan.

Файл: `backend/main.ts`

```typescript
app.use(morgan("combined"));
```

## 3. Файлове логування подій

Winston логер записує повідомлення у файл `logs/app.log` та виводить у консоль.

Файл: `backend/services/logger.service.ts`

```typescript
import { Injectable, LoggerService as NestLoggerService } from "@nestjs/common";
import { createLogger, format, transports } from "winston";
import * as fs from "fs";
import * as path from "path";

@Injectable()
export class LoggerService implements NestLoggerService {
  private readonly logFile = path.join(process.cwd(), "logs", "app.log");
  private readonly logger;

  constructor() {
    const logsDir = path.dirname(this.logFile);
    if (!fs.existsSync(logsDir)) {
      fs.mkdirSync(logsDir, { recursive: true });
    }

    this.logger = createLogger({
      level: "info",
      format: format.combine(
        format.timestamp({ format: "YYYY-MM-DD HH:mm:ss" }),
        format.printf(({ timestamp, level, message, stack }) => {
          return `${timestamp} ${level.toUpperCase()}: ${message}${stack ? `\n${stack}` : ""}`;
        }),
      ),
      transports: [
        new transports.Console(),
        new transports.File({ filename: this.logFile, level: "info" }),
      ],
    });
  }

  log(message: string) {
    this.logger.info(message);
  }

  error(message: string, trace?: string) {
    this.logger.error(message, { stack: trace });
  }

  warn(message: string) {
    this.logger.warn(message);
  }

  debug(message: string) {
    this.logger.debug(message);
  }

  verbose(message: string) {
    this.logger.verbose(message);
  }

  logInfo(message: string) {
    this.log(message);
  }

  logError(message: string, error?: any) {
    this.error(message, error instanceof Error ? error.stack : String(error));
  }
}
```

## 4. Обробка помилок

Глобальний фільтр перехоплює помилки, логувує їх через Winston та повертає JSON відповідь.

Файл: `backend/filters/all-exceptions.filter.ts`

```typescript
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
  HttpStatus,
} from "@nestjs/common";
import { Request, Response } from "express";
import { LoggerService } from "../services/logger.service";

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  constructor(private readonly logger: LoggerService) {}

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const errorResponse =
      exception instanceof HttpException
        ? exception.getResponse()
        : "Internal server error";

    const errorMessage =
      exception instanceof Error ? exception.stack : JSON.stringify(exception);

    this.logger.logError(
      `HTTP ${request.method} ${request.url} ${status}`,
      errorMessage,
    );

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      error: errorResponse,
    });
  }
}
```

Інтеграція у `backend/main.ts`:

```typescript
app.useGlobalFilters(new AllExceptionsFilter(app.get(LoggerService)));
```

## 5. Завантаження одного файлу

Ендпоінт `/upload` реалізовано одним файлом через `FileInterceptor`.

Файл: `backend/controllers/upload.controller.ts`

```typescript
import {
  BadRequestException,
  Controller,
  Post,
  UploadedFile,
  UseInterceptors,
} from "@nestjs/common";
import {
  ApiBody,
  ApiConsumes,
  ApiOperation,
  ApiResponse,
  ApiTags,
} from "@nestjs/swagger";
import { FileInterceptor } from "@nestjs/platform-express";
import { diskStorage } from "multer";
import * as fs from "fs";
import * as path from "path";

const UPLOAD_DIR = path.join(process.cwd(), "uploads");
if (!fs.existsSync(UPLOAD_DIR)) {
  fs.mkdirSync(UPLOAD_DIR, { recursive: true });
}

const storage = diskStorage({
  destination: UPLOAD_DIR,
  filename: (_req, file, callback) => {
    const uniqueSuffix = `${Date.now()}-${Math.round(Math.random() * 1e9)}`;
    callback(null, `${uniqueSuffix}-${file.originalname}`);
  },
});

const fileFilter = (_req, file, callback) => {
  const allowedMimeTypes = ["image/jpeg", "image/png", "application/pdf"];
  if (!allowedMimeTypes.includes(file.mimetype)) {
    return callback(
      new BadRequestException("Дозволені типи файлів: jpg, png, pdf"),
      false,
    );
  }
  callback(null, true);
};

const uploadOptions = {
  storage,
  fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024,
  },
};

@ApiTags("upload")
@Controller("upload")
export class UploadController {
  @Post()
  @ApiOperation({ summary: "Upload a single file" })
  @ApiConsumes("multipart/form-data")
  @ApiBody({
    schema: {
      type: "object",
      properties: {
        file: { type: "string", format: "binary" },
      },
    },
  })
  @ApiResponse({ status: 201, description: "File uploaded." })
  @UseInterceptors(FileInterceptor("file", uploadOptions))
  uploadFile(@UploadedFile() file: Express.Multer.File) {
    if (!file) {
      throw new BadRequestException("Файл не завантажено");
    }
    return {
      filename: file.filename,
      originalName: file.originalname,
      path: file.path,
      size: file.size,
      mimetype: file.mimetype,
    };
  }
}
```

## 6. Завантаження кількох файлів

Підтримка множинного завантаження додана у тому ж контролері.

```typescript
import { FilesInterceptor } from "@nestjs/platform-express";

  @Post("/multiple")
  @ApiOperation({ summary: "Upload multiple files" })
  @ApiConsumes("multipart/form-data")
  @ApiBody({
    schema: {
      type: "object",
      properties: {
        files: {
          type: "array",
          items: { type: "string", format: "binary" },
        },
      },
    },
  })
  @ApiResponse({ status: 201, description: "Files uploaded." })
  @UseInterceptors(FilesInterceptor("files", 10, uploadOptions))
  uploadMultiple(@UploadedFiles() files: Express.Multer.File[]) {
    if (!files || files.length === 0) {
      throw new BadRequestException("Файли не завантажено");
    }
    return files.map((file) => ({
      filename: file.filename,
      originalName: file.originalname,
      path: file.path,
      size: file.size,
      mimetype: file.mimetype,
    }));
  }
```

## 7. Валідація файлів

Перевірка MIME-типу та обмеження розміру реалізовані в `uploadOptions`.

```typescript
const allowedMimeTypes = ["image/jpeg", "image/png", "application/pdf"];

const uploadOptions = {
  storage,
  fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024,
  },
};
```

Це забезпечує:

- дозволені типи файлів: `jpg`, `png`, `pdf`
- макс. розмір одного файлу 5 МБ
- збереження у папку `uploads`

## 8. Моніторинг стану сервера

Ендпоінт `/status` повертає uptime і пам’ять.

Файл: `backend/controllers/system.controller.ts`

```typescript
import { Controller, Get } from "@nestjs/common";
import { ApiOperation, ApiResponse, ApiTags } from "@nestjs/swagger";

@ApiTags("system")
@Controller()
export class SystemController {
  @Get("status")
  @ApiOperation({ summary: "Get server status" })
  @ApiResponse({ status: 200, description: "Server status returned." })
  getStatus() {
    return {
      uptime: process.uptime(),
      memoryUsage: process.memoryUsage(),
      timestamp: new Date().toISOString(),
    };
  }
}
```

## 9. Вимірювання часу відповіді

Middleware `ResponseTimeMiddleware` обчислює час запиту і логує його.

Файл: `backend/middleware/response-time.middleware.ts`

```typescript
import { Injectable, NestMiddleware } from "@nestjs/common";
import { NextFunction, Request, Response } from "express";
import { LoggerService } from "../services/logger.service";

@Injectable()
export class ResponseTimeMiddleware implements NestMiddleware {
  constructor(private readonly logger: LoggerService) {}

  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();
    res.on("finish", () => {
      const duration = Date.now() - start;
      this.logger.logInfo(
        `Request ${req.method} ${req.originalUrl} completed in ${duration}ms`,
      );
    });
    next();
  }
}
```

Реєстрація в `backend/app.module.ts`:

```typescript
import { Module, MiddlewareConsumer, NestModule } from "@nestjs/common";
import { ResponseTimeMiddleware } from "./middleware/response-time.middleware";

@Module({
  // ...
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(ResponseTimeMiddleware).forRoutes("*");
  }
}
```

## 10. Інтеграція менеджера процесів

PM2 конфігурація для запуску NestJS-застосунку.

Файл: `backend/ecosystem.config.js`

```js
module.exports = {
  apps: [
    {
      name: "lab-adaptive-backend",
      script: "pnpm",
      args: "exec ts-node main.ts",
      cwd: __dirname,
      env: {
        NODE_ENV: "production",
      },
    },
  ],
};
```

Скрипти у `backend/package.json`:

```json
"scripts": {
  "pm2:start": "pm2 start ecosystem.config.js",
  "pm2:restart": "pm2 restart lab-adaptive-backend",
  "pm2:stop": "pm2 stop lab-adaptive-backend",
  "pm2:logs": "pm2 logs lab-adaptive-backend"
}
```

Для перевірки:

- `pnpm --filter backend exec pm2 start ecosystem.config.js`
- `pnpm --filter backend exec pm2 logs lab-adaptive-backend`
- `pnpm --filter backend exec pm2 restart lab-adaptive-backend`

---




