# ClassLoayza

Guía técnica e instructivo detallado paso a paso para la instalación de Node.js, configuración de variables de entorno en Windows, aprovisionamiento de PostgreSQL y pgAdmin 4, inicialización de un monorrepósito con Nx Workspace, generación de una API en NestJS e integración del ORM Prisma.

---

## 1. Descargar e Instalar Node.js (2 Métodos)

Node.js es el entorno de ejecución para JavaScript en el servidor. Necesitamos la versión LTS (Long Term Support) para garantizar compatibilidad y estabilidad.

### Método 1: Por Interfaz Gráfica (Instalador Web)
1. Abra su navegador web y acceda a [https://nodejs.org/](https://nodejs.org/).
2. Haga clic en el botón que indica **LTS** (por ejemplo, `20.x.x LTS` o superior) para descargar el instalador `.msi` para Windows.
3. Busque el archivo descargado en su carpeta de Descargas y ejecútelo con doble clic.
4. En el asistente de instalación:
   * Haga clic en **Next** (Siguiente).
   * Acepte los términos de la licencia y haga clic en **Next**.
   * Deje la ruta de instalación por defecto (`C:\Program Files\nodejs\`) y haga clic en **Next**.
   * En la pantalla de selección de componentes, asegúrese de que la opción **"Add to PATH"** tenga el icono de disco duro (instalado por defecto). Haga clic en **Next**.
   * Opcional: En la pantalla de herramientas adicionales, puede marcar la casilla para instalar automáticamente herramientas de compilación de C/C++ (necesario solo para compilar algunos módulos nativos pesados).
   * Haga clic en **Install** (Instalar).
   * Si Windows solicita permisos de administrador (UAC), haga clic en **Sí**.
5. Haga clic en **Finish** al finalizar la instalación.

### Método 2: Por Línea de Comandos (Windows Package Manager - Winget)
Si prefiere instalar software sin usar el navegador, Windows incluye un gestor de paquetes oficial (`winget`) ejecutable desde la terminal:
1. Presione la tecla `Windows` de su teclado, escriba `cmd` o `PowerShell`, haga clic derecho sobre el icono y seleccione **"Ejecutar como administrador"**.
2. Ejecute el siguiente comando para buscar y descargar automáticamente la versión LTS oficial de Node.js:
   ```powershell
   winget install OpenJS.NodeJS.LTS
   ```
3. Acepte los términos escribiendo `Y` (o `S` si su terminal está en español) y presionando `Enter`. El sistema descargará e instalará Node.js de forma silenciosa en segundo plano.
4. Cierre la terminal activa y abra una nueva para actualizar el entorno.

---

## 2. Agregar Node.js y npm al PATH de Windows (2 Métodos)

El `PATH` es una variable de entorno del sistema que le indica a Windows en qué carpetas buscar los ejecutables de los comandos que escribes en la consola. Si al escribir `node` o `npm` obtienes el error *"no se reconoce como un comando interno o externo"*, debes agregar las rutas manualmente.

### Método 1: Por Interfaz Gráfica (GUI de Windows)
1. Presione la combinación de teclas `Windows + R` en su teclado para abrir el cuadro de diálogo **Ejecutar**.
2. Escriba `sysdm.cpl` y presione `Enter` (esto abrirá la ventana de Propiedades del Sistema).
3. Haga clic en la pestaña superior **Opciones avanzadas** (o *Advanced*).
4. Haga clic en el botón inferior llamado **Variables de entorno...** (*Environment Variables*).
5. En la ventana que aparece, verá dos secciones. Busque en la sección superior (**Variables de usuario para [TuUsuario]**) o inferior (**Variables del sistema**) una variable llamada `Path` o `PATH`.
6. Selecciónela con un clic y presione el botón **Editar...** (*Edit*).
7. En la lista de rutas, verifique si existen estas carpetas. Si no están, agregue cada una haciendo clic en el botón **Nuevo** (*New*):
   * Para los binarios de Node: `C:\Program Files\nodejs\`
   * Para los comandos globales instalados con npm: `%USERPROFILE%\AppData\Roaming\npm`
8. Presione **Aceptar** en la ventana de edición.
9. Presione **Aceptar** en la ventana de Variables de Entorno.
10. Presione **Aplicar** y **Aceptar** en la ventana de Propiedades del Sistema.
11. Cierre todas las consolas abiertas. Abra una nueva consola y verifique escribiendo:
    ```bash
    node -v
    npm -v
    ```

### Método 2: Por Línea de Comandos (PowerShell)
Puede agregar rutas a su PATH de usuario de manera inmediata ejecutando un script de PowerShell:
1. Abra **PowerShell** (no requiere privilegios de administrador si modifica las variables de usuario).
2. Ejecute el siguiente bloque de comandos para agregar las rutas necesarias en caso de que no existan previamente:
   ```powershell
   # 1. Obtener la variable Path del usuario actual
   $oldPath = [Environment]::GetEnvironmentVariable("Path", "User")

   # 2. Definir las rutas a agregar
   $nodePath = "C:\Program Files\nodejs"
   $npmPath = "$env:USERPROFILE\AppData\Roaming\npm"

   # 3. Validar y concatenar si no están presentes
   $updatedPath = $oldPath
   if ($oldPath -notlike "*$nodePath*") {
       $updatedPath = "$updatedPath;$nodePath"
   }
   if ($oldPath -notlike "*$npmPath*") {
       $updatedPath = "$updatedPath;$npmPath"
   }

   # 4. Guardar de nuevo en el registro del sistema de forma permanente
   [Environment]::SetEnvironmentVariable("Path", $updatedPath, "User")
   ```
3. Cierre y vuelva a abrir su consola de PowerShell para que los cambios surtan efecto. Verifique la versión instalada:
   ```powershell
   node -v
   npm -v
   ```

---

## 3. Descargar e Instalar pgAdmin 4 y PostgreSQL (2 Métodos)

PostgreSQL es el motor de base de datos relacional y pgAdmin 4 es la interfaz gráfica de administración oficial para gestionarlo de forma amigable.

### Método 1: Por Interfaz Gráfica (Instalador Completo Web)
1. Ingrese al sitio oficial de descargas de PostgreSQL para Windows: [https://www.postgresql.org/download/windows/](https://www.postgresql.org/download/windows/).
2. Haga clic en el enlace **"Download the installer"** (Descargar el instalador) proporcionado por la empresa EDB.
3. Seleccione la versión más reciente compatible (por ejemplo, PostgreSQL `16` o `17` para Windows x86-64) y haga clic en **Download**.
4. Ejecute el instalador `.exe` descargado como Administrador (clic derecho -> Ejecutar como Administrador).
5. En el instalador:
   * **Carpeta de instalación:** Deje la por defecto (`C:\Program Files\PostgreSQL\16` o similar) y presione **Next**.
   * **Selección de componentes:** Asegúrese de tener marcadas las casillas de:
     * **PostgreSQL Server** (El motor principal).
     * **pgAdmin 4** (La interfaz gráfica de administración).
     * **Command Line Tools** (Herramientas de consola como `psql`).
   * **Directorio de datos:** Déjelo por defecto y haga clic en **Next**.
   * **Contraseña del Superusuario:** Ingrese una contraseña segura para el usuario maestro administrador llamado `postgres`. **Importante:** Escriba y memorice esta contraseña, la necesitará para acceder al panel.
   * **Puerto:** Mantenga el puerto por defecto `5432` y haga clic en **Next**.
   * **Configuración regional:** Selecciónela por defecto y haga clic en **Next**.
   * Presione **Next** en las pantallas de confirmación e inicie la instalación.
6. Al finalizar, desmarque la casilla de lanzar **Stack Builder** (no es necesario para desarrollo básico) y presione **Finish**.

### Método 2: Por Línea de Comandos (Winget)
Si desea realizar una instalación de servidor local rápida y aislada usando la terminal:
1. Abra una terminal de **PowerShell como Administrador**.
2. Instale el motor de base de datos PostgreSQL mediante el siguiente comando:
   ```powershell
   winget install PostgreSQL.PostgreSQL
   ```
3. A continuación, instale el administrador visual independiente pgAdmin 4:
   ```powershell
   winget install Postgres.pgAdmin
   ```
4. Siga las instrucciones en pantalla provistas por el instalador nativo de Windows. Al terminar la descarga, el servicio de PostgreSQL se ejecutará automáticamente de fondo.

---

## 4. Configurar Base de Datos y Roles en pgAdmin 4 (2 Métodos)

Necesitamos crear un rol de inicio de sesión seguro (`jala_admin`), configurar su contraseña y asignarle una base de datos propia (`jala_db`) de la cual sea dueño.

### Método 1: Por Interfaz Gráfica (pgAdmin 4 GUI)
1. Busque y abra la aplicación **pgAdmin 4** desde el menú de Inicio de Windows.
2. Al iniciar por primera vez, pgAdmin 4 le pedirá configurar una **Master Password**. Configure una clave sencilla y recuérdela (se usa para proteger sus credenciales guardadas).
3. En el panel izquierdo, expanda el menú de **Servers** (Servidores).
4. Haga clic en **PostgreSQL 16** (o la versión instalada). El sistema le pedirá la contraseña del superusuario administrador (`postgres`) que configuró en el proceso de instalación anterior. Escríbala y marque la opción "Save password" para no tener que ingresarla continuamente.
5. Una vez conectado, haga clic derecho sobre el servidor **PostgreSQL** en la barra lateral y seleccione **Query Tool** (Herramienta de consultas) en el menú contextual.
6. Copie el siguiente script SQL y péguelo en el editor de consultas en blanco:
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
7. Presione la tecla **F5** de su teclado o haga clic en el botón de reproducción/rayo (`Execute/Refresh`) en la barra de herramientas superior para ejecutar la consulta.
8. Verifique en el panel inferior que se muestre el mensaje: `Query returned successfully`. En la barra lateral izquierda, haga clic derecho en **Databases** y seleccione **Refresh** para confirmar la existencia de `jala_db`.

### Método 2: Por Línea de Comandos (consola interactiva psql)
Si prefiere administrar PostgreSQL directo desde su terminal de comandos:
1. Abra su consola (CMD o PowerShell).
2. Conéctese localmente al servidor utilizando el usuario maestro `postgres`:
   ```bash
   psql -U postgres -h localhost
   ```
3. Ingrese la contraseña de administrador maestro cuando la consola la solicite.
4. Escriba o pegue el siguiente bloque SQL línea por línea en la consola interactiva (cada sentencia SQL debe terminar con un punto y coma `;`):
   ```sql
   CREATE ROLE jala_admin WITH LOGIN PASSWORD 'Jala_2024_Secur3!';
   CREATE DATABASE jala_db WITH OWNER = jala_admin ENCODING = 'UTF8' CONNECTION LIMIT = -1;
   GRANT ALL PRIVILEGES ON DATABASE jala_db TO jala_admin;
   ```
5. Escriba `\q` y presione `Enter` para desconectarse y salir de la consola interactiva.

---

## 5. Inicialización de Nx Workspace y NestJS

Nx permite estructurar un monorrepósito rápido. Crearemos la aplicación central de backend (`api-core`) sobre él.

### Paso 1: Crear el espacio de trabajo Nx
Abra la consola de comandos en la carpeta raíz de sus proyectos (por ejemplo, `C:\proyectos\`) y ejecute:
```bash
npx create-nx-workspace@latest jala-workspace
```
Configure las opciones interactivas con los siguientes valores exactos:
* **Where would you like to create your workspace?:** `jala-workspace`
* **Which stack would you like to use?:** `Node` (o `None` si desea configuraciones personalizadas manuales)
* **What framework would you like to use?:** `None`
* **Integrated monorepo, or standalone project?:** `Integrated monorepo` (para soportar múltiples aplicaciones)
* **Would you like to configure a CI pipeline...?:** `Skip`

Una vez finalizado, ingrese al directorio:
```bash
cd jala-workspace
```

### Paso 2: Agregar el soporte de NestJS a Nx
Descargue el plugin oficial de generación de aplicaciones de NestJS de forma local:
```bash
npm install -D @nx/nest
```

### Paso 3: Generar la aplicación de NestJS
Ejecute el comando para estructurar la API central:
```bash
npx nx g @nx/nest:application api-core
```
Esto estructurará la base de código de la aplicación de NestJS dentro del directorio `apps/api-core`.

---

## 6. Integración y Configuración del ORM Prisma

Prisma nos proveerá de un cliente de base de datos estructurado y tipado.

### Paso 1: Instalar dependencias
Instale Prisma CLI para desarrollo y el Prisma Client ejecutable para las consultas:
```bash
npm install -D prisma
npm install @prisma/client
```

### Paso 2: Inicializar Prisma
Cree los archivos de configuración base de Prisma:
```bash
npx prisma init
```
Esto creará el directorio `/prisma` con el archivo `schema.prisma` y un archivo `.env` en la raíz de su espacio de trabajo.

### Paso 3: Configurar la URL de conexión
Abra el archivo `.env` en su editor de código. Modifique la variable `DATABASE_URL` especificando las credenciales del rol y base de datos configuradas en pgAdmin:
```env
DATABASE_URL="postgresql://jala_admin:Jala_2024_Secur3!@localhost:5432/jala_db?schema=public"
```

### Paso 4: Definir el Schema de Base de Datos
Abra `prisma/schema.prisma` y configure el proveedor de base de datos a `postgresql` y defina un modelo inicial de prueba:
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

### Paso 5: Generar y Aplicar las Migraciones
Para mapear la estructura declarada en su modelo hacia tablas físicas dentro de PostgreSQL, ejecute en su terminal:
```bash
npx prisma migrate dev --name init
```
Este comando creará la tabla `User` en su base de datos `jala_db` y compilará las interfaces de TypeScript con tipado estático seguro dentro del directorio `node_modules/.prisma`.

---

## 7. Configuración e Inyección del ORM Prisma en NestJS (`api-core`)

Debemos exponer el cliente autogenerado de Prisma como un servicio inyectable global dentro de NestJS.

### Paso 1: Crear el Prisma Service
Cree el archivo de servicio de conexión en `apps/api-core/src/app/prisma.service.ts` y escriba la siguiente clase:
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
Abra `apps/api-core/src/app/app.module.ts`, importe el servicio e inyéctelo dentro del arreglo de `providers`:
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

Ahora, la conexión de alto rendimiento con PostgreSQL está disponible en su backend listo para producción.
