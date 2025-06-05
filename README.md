# Guía de Configuración de Aplicación Web Docker para Windows Server 2022

Este documento proporciona una guía paso a paso para configurar y ejecutar una aplicación web PHP en Docker sobre Windows Server 2022, utilizando XAMPP para MySQL.

## Tabla de Contenidos

1. [Requisitos Previos](#requisitos-previos)
2. [Instalación de Docker Engine](#instalación-de-docker-engine)
3. [Configuración del Proyecto](#configuración-del-proyecto)
4. [Estructura del Proyecto](#estructura-del-proyecto)
5. [Construcción y Ejecución](#construcción-y-ejecución)
6. [Configuración de la Base de Datos](#configuración-de-la-base-de-datos)
7. [Solución de Problemas](#solución-de-problemas)

## Requisitos Previos

- Windows Server 2022
- Acceso de administrador al servidor
- Conexión a Internet
- Mínimo 4GB de RAM
- 20GB de espacio en disco libre
- XAMPP instalado (para MySQL)

## Instalación de Docker Engine

1. **Abrir PowerShell como Administrador**

2. **Descargar e instalar Docker CE**

   ```powershell
   Invoke-WebRequest -UseBasicParsing "https://raw.githubusercontent.com/microsoft/Windows-Containers/Main/helpful_tools/Install-DockerCE/install-docker-ce.ps1" -o install-docker-ce.ps1
   .\install-docker-ce.ps1
   ```

3. **Reiniciar el servidor**

   ```powershell
   Restart-Computer
   ```

4. **Verificar la instalación**
   ```powershell
   docker --version
   ```

## Configuración del Proyecto

1. **Crear la red de Docker**

   ```powershell
   docker network create [nombre_de_la_red]
   ```

2. **Preparar la estructura de directorios**
   ```
   granja_docker/
   ├── src/                    # Código fuente de la aplicación
   ├── php/                    # Archivos de PHP (instalados manualmente)
   ├── Dockerfile.windows      # Configuración de la imagen Docker
   ├── docker-compose.yml      # Configuración de servicios
   └── database.php           # Configuración de la base de datos
   ```

## Estructura del Proyecto

### Dockerfile.windows

El Dockerfile está configurado para:

- Usar Windows Server 2022 como imagen base
- Instalar IIS y configurar PHP manualmente
- Configurar las extensiones necesarias de PHP
- Exponer el puerto 8080

```dockerfile
# Instalar IIS y herramientas de administración
RUN powershell -Command \
    Install-WindowsFeature Web-Server -IncludeManagementTools

# Configurar PHP (instalado manualmente)
COPY ./php/ C:/php/

# Configurar extensiones PHP
RUN echo "extension=mysqli" >> C:\php\php.ini && \
    echo "extension=pdo_mysql" >> C:\php\php.ini
```

### docker-compose.yml

Configura el servicio web con:

- Mapeo de puertos (8081:8080)
- Volúmenes para el código fuente
- Variables de entorno para la base de datos
- Conexión a la red personalizada

```yaml
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.windows
    ports:
      - "8081:8080"
    volumes:
      - ./src:C:\inetpub\wwwroot
    environment:
      - DB_HOST=host.docker.internal
      - DB_USER=root
      - DB_PASS=Yoisi_palacios20
      - DB_NAME=granja
```

### database.php

Configura la conexión a MySQL en XAMPP:

```php
$host = 'host.docker.internal';  # Accede al host desde el contenedor
$dbname = 'nombre_base_de_datos';
$user = 'tu_usuario';
$pass = 'tu_contraseña';

$conn = new mysqli($host, $user, $pass, $dbname);
```

## Construcción y Ejecución

1. **Construir la imagen**

   ```powershell
   docker-compose build
   ```

2. **Iniciar los contenedores**

   ```powershell
   docker-compose up -d
   ```

3. **Verificar el estado**

   ```powershell
   docker-compose ps
   ```

4. **Acceder a la aplicación**
   - Abrir un navegador web
   - Navegar a `http://localhost:8081`

## Configuración de la Base de Datos

1. **Asegurarse de que XAMPP esté instalado y MySQL esté ejecutándose**

   - Abrir XAMPP Control Panel
   - Iniciar el servicio MySQL

2. **Crear la base de datos**

   ```sql
   CREATE DATABASE nombre_de_tu_base_datos;
   ```

3. **Configurar las credenciales en XAMPP**
   - Usuario: tu_usuario
   - Contraseña: tu_contraseña
   - Base de datos: tu_base_de_datos

## Solución de Problemas

### Problemas Comunes

1. **El contenedor no inicia**

   ```powershell
   docker-compose logs
   ```

2. **Problemas de conexión a la base de datos**

   - Verificar que XAMPP MySQL esté ejecutándose
   - Comprobar las credenciales en database.php
   - Asegurarse de que el puerto de MySQL (3306) esté accesible

3. **Problemas de permisos**
   ```powershell
   icacls "C:\inetpub\wwwroot" /grant "IIS_IUSRS:(OI)(CI)F"
   ```

### Comandos Útiles

- **Ver registros del contenedor**

  ```powershell
  docker-compose logs -f
  ```

- **Reiniciar el contenedor**

  ```powershell
  docker-compose restart
  ```

- **Detener y eliminar contenedores**
  ```powershell
  docker-compose down
  ```

## Notas Importantes

1. **Configuración de PHP**

   - PHP se instala manualmente desde el archivo ZIP
   - Las extensiones necesarias se configuran en php.ini
   - El directorio php/ contiene los archivos de PHP

2. **Configuración de MySQL**

   - Se utiliza XAMPP para MySQL
   - La conexión se realiza a través de host.docker.internal
   - El puerto por defecto es 3306

3. **Puertos**
   - Aplicación web: 8081 (host) -> 8080 (contenedor)
   - MySQL: 3306 (XAMPP)

## Mantenimiento

1. **Actualizar la imagen**

   ```powershell
   docker-compose pull
   docker-compose up -d
   ```

2. **Limpiar recursos no utilizados**

   ```powershell
   docker system prune
   ```

3. **Realizar copias de seguridad**
   - Realizar backup regular de la base de datos desde XAMPP
   - Mantener copias de seguridad del código fuente

## Soporte

Para reportar problemas o solicitar ayuda:

1. Revisar los registros del contenedor
2. Verificar la configuración de XAMPP y MySQL
3. Comprobar los permisos de los archivos
4. Asegurarse de que todos los servicios estén funcionando correctamente


Luego de haber usado todos estos pasos y ejecutar el servidor debe aparecer de la siguiente manara en el terminal 

![image](https://github.com/user-attachments/assets/f1556143-43c7-4db7-865d-efc48103d2d0)
![image](https://github.com/user-attachments/assets/b7237038-f997-44b1-b22c-07a49b731dde)


