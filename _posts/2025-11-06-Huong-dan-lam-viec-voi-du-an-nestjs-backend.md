---
title: "Hướng dẫn làm việc với dự án NestJS Backend"
date: 2025-11-06 09:40:00  +0700
categories: [Docs, NestJS]
tags: [docs, nestjs]
---

> _"Hướng dẫn này chỉ áp dụng khi bạn làm trong dự án do mình xây dựng cấu trúc dự án."_

## **1. Hướng dẫn chạy**

Trước khi bắt đầu, hãy đảm bảo rằng bạn đã cài đặt:

- **Node.js** phiên bản 18 trở lên
- **npm** (đi kèm Node.js). Bạn cũng có thể dùng **pnpm** hoặc
  **yarn** nếu quen thuộc
- **Docker** (để chạy các dịch vụ phụ trợ như PostgreSQL, Keycloak,
  v.v.)
- **VS Code** hoặc một trình soạn thảo có hỗ trợ TypeScript và ESLint

Nếu máy bạn đã sẵn sàng với các yêu cầu trên, hãy đến với bước cài đặt môi trường cục bộ:

1.  Clone dự án về máy:

    ```bash
    git clone https://github.com/<github-username>/<project-name>.git
    cd <project-name>
    ```

2.  Cài đặt các thư viện cần thiết:

    ```bash
    npm install
    ```

3.  Chạy các dịch vụ nền bằng Docker:

    ```bash
    cd docker
    docker compose up -d
    ```

4.  Khởi tạo cơ sở dữ liệu và dữ liệu mẫu:

    ```bash
    npm run migration:run
    npm run seed:run
    ```

5.  Khởi động dự án ở chế độ development:

    ```bash
    npm run dev
    ```

    > Ứng dụng NestJS sẽ chạy ở cổng được cấu hình trong file
    > `config/local.js`.

## **2. Một số lệnh hữu ích**

| Lệnh               | Chức năng                                         |
| ------------------ | ------------------------------------------------- |
| `npm run lint`     | Kiểm tra lỗi code theo chuẩn ESLint               |
| `npm run test`     | Chạy toàn bộ test đơn vị _(unit tests)_           |
| `npm run test:e2e` | Chạy test đầu cuối _(end-to-end)_                 |
| `npm run build`    | Biên dịch dự án thành mã JavaScript để triển khai |

> 💡 Mẹo: Bạn có thể dùng `npm run test --watch` để tự động chạy test
> khi code thay đổi.

<br>
Nếu bạn muốn bật **Keycloak** cho môi trường phát triển (xác thực người
dùng), chạy thêm lệnh (mặc định nếu không thay đổi gì ở file docker-compose.yml thì không cần chạy lệnh dưới):

```bash
docker compose up -d keycloak-db keycloak
```

Sau đó vào trình duyệt và truy cập: `http://localhost:8080` (tài khoản mặc định: admin / admin)

> Nếu bạn chỉ gõ `docker compose up -d` (không thêm gì sau đó), Docker sẽ chạy **tất cả** các service trong docker-compose.yml. Nếu bạn chỉ định cụ thể (như `keycloak-db keycloak`), Docker **chỉ khởi động** hai service đó — hữu ích khi bạn muốn tiết kiệm tài nguyên hoặc chỉ cần Keycloak mà không cần chạy toàn bộ hệ thống.

## **3. Cách để thay đổi cấu hình mặc định ( port, token sercet,... )**

Hệ thống **NestJS Backend** hỗ trợ cấu hình linh hoạt thông qua các
file trong thư mục `config/` và các biến môi trường (`.env`).\
Điều này giúp bạn dễ dàng thay đổi **port**, **thông tin cơ sở dữ
liệu**, hoặc **token secret** mà không cần chỉnh sửa trực tiếp mã nguồn.

Cấu trúc thư mục `config/`:

```
 config/
    ├── custom-environment-variables.js
    ├── default.js
    ├── local.js
    └── test.js
```

File `custom-environment-variables.js` định nghĩa cách các biến môi
trường ánh xạ vào cấu hình nội bộ.

Ví dụ:

```js
port: number('PORT'),
core: {
  database: {
    host: 'CORE_DATABASE_HOST',
    port: number('CORE_DATABASE_PORT'),
  },
}
```

Nếu bạn tạo file `.env` như sau:

```bash
PORT=4000
CORE_DATABASE_HOST=localhost
CORE_DATABASE_PORT=5432
```

Thì ứng dụng sẽ đọc ra đúng các giá trị tương ứng.

```javascript
console.log(getConfig("core.database.port")); // 5432
```

<br>

Có **2 cách** để đổi cấu hình của ứng dụng.

##### 🥇 Cách 1: Dùng file `.env` (Khuyến nghị)

Tạo file `.env` ở thư mục gốc của dự án (cùng cấp với `package.json`):

```bash
PORT=8080
APP_NAME=payment
```

Khi đó, cấu hình trong file `config/custom-environment-variables.js` sẽ
tự động ánh xạ (`map`) biến `PORT` sang `port` trong hệ thống:

```js
port: number("PORT");
```

=\> Kết quả: ứng dụng sẽ khởi chạy ở cổng `8080`.

##### 🥈 Cách 2: Dùng file `config/local.js`

Nếu bạn muốn ghi đè cục bộ (chỉ cho máy cá nhân), có thể tạo hoặc chỉnh
file `config/local.js` như sau:

```js
module.exports = {
  port: 8080,
  appName: "payment",
  core: {
    database: {
      host: "localhost",
      port: 5432,
      username: "postgres",
      password: "123456",
      dbName: "payment_db",
    },
  },
};
```

> ⚠️ **Lưu ý quan trọng:**\
> File `local.js` và `.env` **chỉ nên dùng cho máy cá nhân**.\
> Không được push file này lên GitHub vì có thể chứa thông tin nhạy cảm
> (mật khẩu, host, token,...).

## **4. Hướng dẫn tạo Migration và Seed**

Trong dự án **NestJS Backend**, thư mục `db/` quản lý toàn bộ cấu trúc
và dữ liệu khởi tạo của cơ sở dữ liệu (Database).\
Cấu trúc chung như sau:

```
    db/
    ├── migrations/          # Lưu các file migration (thay đổi cấu trúc CSDL)
    ├── seeds/               # Lưu các file seed (dữ liệu mẫu, dữ liệu khởi tạo)
    ├── migration.config.ts  # Cấu hình chạy migration
    └── seed.config.ts       # Cấu hình chạy seed
```

Cả hai file `migration.config.ts` và `seed.config.ts` đều dựa trên cấu
hình trong `config/` để kết nối cơ sở dữ liệu.

Ví dụ (trích từ `migration.config.ts`):

```ts
import * as config from "config";
import { DataSource, DataSourceOptions } from "typeorm";

const dataSourceOptions: DataSourceOptions = {
  type: config.get("core.database.type") as any,
  host: config.get("core.database.host") as string,
  port: config.get("core.database.port") as number,
  username: config.get("core.database.username") as string,
  password: config.get("core.database.password") as string,
  database: config.get("core.database.dbName") as string,
  entities: ["src/**/*.entity{.ts,.js}"],
  migrations: ["db/migrations/*{.ts,.js}"],
  synchronize: false,
};

export const connectionSource = new DataSource(dataSourceOptions);
```

> 💡 Cấu hình này đảm bảo rằng mọi thông tin kết nối được lấy tự động từ
> hệ thống `config/` (bao gồm `.env` và `local.js`).

Trong file `migration.config.ts`, có dòng cấu hình rất quan trọng:

```ts
entities: ['src/**/*.entity{.ts,.js}'],
```

Dòng này nói với **TypeORM** rằng:

> "Hãy tự động quét tất cả các file trong thư mục `src/` (và các thư mục
> con) có tên kết thúc bằng `.entity.ts` hoặc `.entity.js`, và coi chúng
> là các **Entity** của hệ thống."\

Nghĩa là khi bạn thêm 1 file đặt tên `.entity.{ts,js}`
hoặc sửa file nào có tên là `.entity.{ts,js}` mà thao tác sửa đó có ảnh hưởng đến cấu hình bảng
thì khi bạn chạy `migration:generate` nó sẽ cập nhật thay đổi đó vào các file migration.
Nói cách khác, bạn **chỉ cần** đặt tên file đúng chuẩn là:

```
user.entity.ts
notification.entity.ts
message.entity.ts
```

...thì TypeORM **tự động nhận diện** chúng khi chạy migration hoặc khởi
tạo kết nối cơ sở dữ liệu --- **không cần đăng ký thủ công từng entity**
nữa 🎯

#### A. Tạo và chạy Migration

Migration dùng để **quản lý thay đổi cấu trúc CSDL** (tạo bảng, thêm
cột, sửa kiểu dữ liệu,...).

##### **Tạo migration mới**

Chạy lệnh:

```bash
npm run migration:generate --name=CreateUserTable
```

Hoặc trực tiếp dùng CLI TypeORM:

```bash
npx typeorm migration:generate db/migrations/CreateUserTable
```

Kết quả: file mới sẽ xuất hiện trong `db/migrations/`:

```
db/migrations/
    └── 1709876543210-CreateUserTable.ts
```

Trong file này, bạn sẽ thấy 2 phương thức:

```ts
import { MigrationInterface, QueryRunner } from "typeorm";

export class CreateUserTable1709876543210 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(
      `CREATE TABLE "user" ("id" SERIAL PRIMARY KEY, "name" VARCHAR(255))`
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP TABLE "user"`);
  }
}
```

<br>

Hoặc bạn cũng có thể tạo 1 file migration rỗng với lệnh:

```bash
npm run migration:create --name=CreateUserTable
```

Kết quả file migration sẽ được tạo như sau:

```ts
import { MigrationInterface, QueryRunner } from "typeorm";

export class CreateUserTable1709876543210 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {}

  public async down(queryRunner: QueryRunner): Promise<void> {}
}
```

Ở đây bạn có thể tùy biến câu lệnh sẽ được áp dụng lên database theo kỹ năng `SQL` của bạn

##### **Chạy migration**

Để áp dụng tất cả migration lên CSDL:

```bash
npm run migration:run
```

Để hoàn tác migration (rollback):

```bash
npm run migration:revert
```

> ⚠️ **Lưu ý:** không chỉnh sửa migration đã chạy trên môi trường
> production. Nếu cần thay đổi, hãy tạo migration mới.

#### B. Tạo và chạy Seed

Seed giúp bạn **thêm dữ liệu mẫu hoặc dữ liệu khởi tạo** (ví dụ tài
khoản admin, danh mục mặc định).

##### **Tạo file seed mới**

Chạy lệnh:

```bash
npm run seed:Create --name=CreateDefaultAdmin
```

Kết quả: file mới sẽ xuất hiện trong `db/seeds/`:

```
db/seeds/
    └── 1709876543210-CreateDefaultAdmin.ts
```

Kết quả file seed sẽ được tạo như sau:

```ts
import * as bcrypt from "bcryptjs";

import { MigrationInterface, QueryRunner } from "typeorm";

export class CreateDefaultAdmin1756903747112 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {}

  public async down(queryRunner: QueryRunner): Promise<void> {}
}
```

Tại đây bạn có thể thoải mái sử dụng câu lệnh `SQL` để có thể tạo các tài khoản mặc định cho hệ thống.  
Ví dụ:

```ts
import * as bcrypt from "bcryptjs";

import { MigrationInterface, QueryRunner } from "typeorm";

export class CreateDefaultAdmin1756903747112 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Check if admin user already exists
    const existingAdmin = await queryRunner.query(
      `SELECT id FROM users WHERE username = $1 OR email = $2`,
      ["admin", "admin@example.com"]
    );

    if (existingAdmin.length > 0) {
      console.log("Admin user already exists");
      return;
    }

    // Insert default admin user
    await queryRunner.query(
      `
            INSERT INTO users (id, username, email, "fullName", role, status, "createdAt", "updatedAt")
            VALUES (
                gen_random_uuid(),
                $1,
                $2,
                $3,
                $4,
                $5,
                NOW(),
                NOW()
            )
        `,
      [
        "admin",
        "admin@example.com",
        "System Administrator",
        "SUPER_ADMIN",
        "ACTIVE",
      ]
    );

    console.log("Default admin user created successfully");
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(
      `DELETE FROM users WHERE username = $1 OR email = $2`,
      ["admin", "admin@example.com"]
    );

    console.log("Default admin user removed");
  }
}
```

##### **Chạy seed**

Chạy lệnh sau:

```bash
npm run seed:run
```

> Lệnh này sử dụng `seed.config.ts` để kết nối database và thực thi các
> seed scripts trong `db/seeds/`.

## **5. Nguyên tắc tách Domain và tổ chức Module**

Dự án **NestJS Backend** được xây dựng theo hướng **Domain-Driven Design (DDD)** kết hợp **CQRS pattern** và **Module-based architecture**.\
Mỗi miền nghiệp vụ (**domain**) được tách thành **một module độc lập**, có thể phát triển, kiểm thử và triển khai riêng mà không ảnh hưởng đến phần khác.

Mục tiêu của kiến trúc này là:

- Giảm sự phụ thuộc giữa các phần trong hệ thống
- Dễ dàng mở rộng tính năng mới mà không làm vỡ cấu trúc cũ
- Hỗ trợ làm việc nhóm (mỗi nhóm phụ trách 1 domain riêng)
- Tối ưu hiệu suất và độ ổn định trong dài hạn
- Có thể scale lên Microservices dễ dàng trong tương lai

#### **A. Cấu trúc tổng thể**

```

libs/
├── core/                      # Core services (database, cache, logger, kafka...)
└── common/                    # Shared code: base classes, utils, constants
src/
├── main.ts                    # Main
├── app.module.ts              # Root module
├── account/                   # Domain 1: Quản lý tài khoản
│   ├── packages/
│   │   ├── account-user/      # Package quản lý người dùng
│   │   ├── account-auth/      # Package xác thực & refresh token
│   │   └── account-shared/    # DTOs, enums, mappers dùng chung
│   └── account.module.ts
└── notification/              # Domain 2: Gửi thông báo
    ├── packages/
    │   ├── notification-sse/  # SSE streaming
    │   └── notification-queue/# Queue (Redis/Kafka)
    └── notification.module.ts
```

<p align="center">
  <img src="/assets/images/docs-nestjs-started/project-strucsture.jpg" alt="Image title_1" />
</p>

#### **B. Nguyên tắc tách domain**

- Domain đại diện cho **một vùng nghiệp vụ riêng** (Account, Notification, Chat, Payment...).\
- Mỗi domain có **module riêng** (`*.module.ts`) chứa controller, service, và package liên quan.

##### **📦 Mỗi domain gồm nhiều package nhỏ theo entity**

Mỗi **package** tương ứng với **một entity hoặc aggregate root**.\
Package chứa toàn bộ logic của entity đó (CQRS, DTO, Mapper, Entity,
Events...).

| Package          | Chức năng                           |
| ---------------- | ----------------------------------- |
| `account-user`   | Quản lý người dùng, hồ sơ, mật khẩu |
| `account-auth`   | Đăng nhập, JWT, refresh token       |
| `account-shared` | DTO, enum, mapper dùng chung        |

> 💡 Mỗi package nên có phạm vi logic nhỏ, dễ kiểm thử và có thể thay thế.

#### **C. Mối quan hệ giữa các Layer**

---

| Layer                       | Vai trò                          | Ví dụ                             |
| --------------------------- | -------------------------------- | --------------------------------- |
| **App Module**              | Tập hợp toàn bộ feature module   | `app.module.ts`                   |
| **Core Module**             | Hạ tầng: DB, Kafka, Cache,Logger | `core/database.module.ts`         |
| **Common Module**           | Base class, types, utils         | `libs/common`                     |
| **Feature Module (Domain)** | Nghiệp vụ chính                  | `src/account`, `src/notification` |
| **Package Layer**           | Nhỏ nhất, ứng với 1 entity       | `account-user`,`account-auth`     |

#### **D. Quy ước khi code domain**

---

| Thành phần    | Quy tắc đặt tên                     | Ví dụ                               |
| ------------- | ----------------------------------- | ----------------------------------- |
| Entity        | Tên file `*.entity.ts`              | `user.entity.ts`                    |
|               | Tên Entity `*Entity` hoặc `Entity*` | `UserEntity`, `EntityUser`          |
| Package       | `<domain>-<entity>`                 | `account-user` hoặc `acc-user`      |
| DTO           | Tên file `*.dto.ts`                 | `login.dto.ts`                      |
|               | Tên Dto `*Dto`                      | `CreateUserDto`,`LoginDto`          |
| Command/Query | Kết thúc bằng `Command`/`Query`     | `CreateUserCommand`,`FindUserQuery` |
| Handler       | Trong thư mục `handlers/`           | `create-user.handler.ts`            |
| Module        | Một domain = một module             | `account.module.ts`                 |

---

> Hãy bắt đầu mọi tính năng mới bằng cách xác định **domain nghiệp
> vụ**, tách domain thành **module riêng**, và chia nhỏ logic thành
> **packages theo entity**. Mỗi module được thiết kế sau này đều có khả năng là 1 service trong hệ thống microservice.

<p align="center">
  <img src="/assets/images/docs-nestjs-started/repo-organization.jpg" alt="Image title_2" />
</p>

## **6. Các kỹ thuật khi code**

> _**Coming soon... Đọc source demo và code theo thoiii**_
