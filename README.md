# ClassLoayza

Guía técnica completa para la instalación de Node.js, configuración del entorno en Windows, inicialización de un monorrepósito empresarial con Nx, generación de una API con NestJS, aprovisionamiento de base de datos en PostgreSQL (pgAdmin 4) e integración del ORM Prisma.

---

## 1. Instalación de Node.js y Configuración del PATH en Windows

Para ejecutar entornos modernos de JavaScript en el servidor (como NestJS o Nx), es necesario instalar el entorno de ejecución Node.js y asegurarse de que sus binarios sean accesibles globalmente desde la terminal.

### Pasos para la instalación:
1. Descargue el instalador oficial de **Node.js LTS** (Long Term Support) desde [nodejs.org](https://nodejs.org/).
2. Ejecute el instalador `.msi` y siga los pasos del asistente. Asegúrese de dejar activada la opción **"Add to PATH"** durante el proceso.

### Agregar manualmente al PATH (en caso de fallos):
Si la terminal (PowerShell o CMD) no reconoce los comandos `node` o `npm` después de la instalación, debe agregar la ruta manualmente a las variables de entorno del sistema:

1. Presione la tecla `Windows`, escriba **"Variables de entorno"** y seleccione **"Editar las variables de entorno del sistema"**.
2. En la ventana de Propiedades del Sistema, haga clic en el botón **"Variables de entorno..."**.
3. En la sección **"Variables de usuario"** (o "Variables del sistema" para todos los usuarios), busque la variable llamada `Path` y haga clic en **"Editar..."**.
4. Haga clic en **"Nuevo"** y añada la ruta exacta donde se instaló Node.js. Por defecto:
   `C:\Program Files\nodejs\`
5. Haga clic en **"Aceptar"** en todas las ventanas.
6. Reinicie su terminal y verifique la correcta instalación ejecutando:
   ```bash
   node -v
   npm -v
   ```

---

## 2. Inicialización del Workspace de Nx

Nx es un framework de compilación inteligente que permite gestionar monorrepósitos con soporte de primera clase para NestJS.

Para crear un nuevo espacio de trabajo vacío o con una estructura base, ejecute en su terminal:

```bash
npx create-nx-workspace@latest jala-workspace
```

Durante el asistente interactivo, seleccione las siguientes opciones:
* **Directory:** `jala-workspace`
* **Stack:** `Node` (o `None` para un monorrepósito vacío)
* **Integrated monorepo vs Standalone:** `Integrated monorepo` (recomendado para múltiples aplicaciones y librerías compartidas)
* **CI pipeline:** `Skip` (o según sus requerimientos de integración continua)

Acceda al directorio recién creado:
```bash
cd jala-workspace
```

---

## 3. Instalación de NestJS y Generación de la API Core

NestJS es un framework progresivo de Node.js para construir aplicaciones eficientes, confiables y escalables en el lado del servidor.

### Paso 1: Instalar el generador de NestJS para Nx
Instale el paquete de integración oficial de NestJS como una dependencia de desarrollo:

```bash
npm install -D @nx/nest
```

### Paso 2: Generar la aplicación de NestJS
Utilice el generador de Nx para crear una nueva aplicación llamada `api-core`:

```bash
npx nx g @nx/nest:application api-core
```

Esto estructurará un directorio `apps/api-core` con los archivos de configuración base de NestJS (módulos, controladores, servicios y archivo `main.ts`).

---

## 4. Configuración de PostgreSQL en pgAdmin 4

Antes de configurar la conexión del backend con la base de datos, debemos inicializar el usuario (rol de base de datos) y la base de datos correspondiente dentro de PostgreSQL.

Abra **pgAdmin 4**, conéctese a su servidor local y abra la herramienta de consultas (**Query Tool**). Ejecute el siguiente script SQL secuencialmente:

```sql
-- 1. Crear el usuario específico para la aplicación (Role con permisos de Login)
CREATE ROLE jala_admin WITH LOGIN PASSWORD 'Jala_2024_Secur3!';

-- 2. Crear la base de datos física y asignar al nuevo usuario como propietario (Owner)
CREATE DATABASE jala_db 
    WITH 
    OWNER = jala_admin
    ENCODING = 'UTF8'
    CONNECTION LIMIT = -1;

-- 3. Conceder todos los privilegios administrativos sobre la base de datos al rol creado
GRANT ALL PRIVILEGES ON DATABASE jala_db TO jala_admin;
```

---

## 5. Integración y Configuración de Prisma ORM

Prisma es un ORM moderno que proporciona un cliente autogenerado y seguro contra tipos (type-safe) a partir de un esquema declarativo.

### Paso 1: Instalar Prisma CLI y el Cliente
Instale la interfaz de comandos de Prisma como dependencia de desarrollo y el cliente de Prisma para consultas en runtime:

```bash
npm install -D prisma
npm install @prisma/client
```

### Paso 2: Inicializar la estructura de Prisma
Ejecute la inicialización de Prisma en su proyecto, indicando que usará PostgreSQL:

```bash
npx prisma init
```

Esto generará:
* Una carpeta `prisma` con el archivo `schema.prisma`.
* Un archivo `.env` en la raíz de su espacio de trabajo.

### Paso 3: Configurar la conexión en el archivo `.env`
Abra el archivo `.env` creado en la raíz y reemplace la variable de entorno `DATABASE_URL` con las credenciales del rol y base de datos configuradas en pgAdmin:

```env
DATABASE_URL="postgresql://jala_admin:Jala_2024_Secur3!@localhost:5432/jala_db?schema=public"
```

### Paso 4: Definir el Modelo de Datos
Abra `prisma/schema.prisma` y defina un modelo de datos inicial a modo de prueba:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
}
```

### Paso 5: Generar y aplicar migraciones
Para crear las tablas correspondientes en PostgreSQL y sincronizar el cliente de Prisma local, ejecute:

```bash
npx prisma migrate dev --name init
```

Este comando:
1. Generará la consulta SQL de migración y la aplicará sobre su base de datos `jala_db`.
2. Generará automáticamente el cliente de Prisma (`PrismaClient`) con tipado estático exacto para su modelo `User`.

---

## 6. Uso de Prisma en NestJS (`api-core`)

Para integrar Prisma dentro de su aplicación de NestJS de forma estructurada, cree un servicio global para manejar la conexión:

### Paso 1: Crear el Prisma Service
Cree un archivo `apps/api-core/src/app/prisma.service.ts`:

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }
}
```

### Paso 2: Registrar el servicio en el módulo principal
Abra `apps/api-core/src/app/app.module.ts` e inyecte su servicio en el arreglo de `providers`:

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { PrismaService } from './prisma.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService, PrismaService],
})
export class AppModule {}
```

Ahora puede inyectar y utilizar `PrismaService` en cualquier controlador o servicio dentro de su API para realizar consultas estructuradas de alto rendimiento.
