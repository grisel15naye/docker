# Procedimiento: Spring Boot + MySQL usando Docker

> Nota de laboratorio / procedimiento paso a paso.  
El orden **no se altera**. Solo se mejora la redacción y claridad.

---

## 1. Abrir el Símbolo del Sistema

Se inicia el proceso abriendo el **Símbolo del sistema / terminal** en el sistema operativo.

![Abrir símbolo de sistema](https://t90131963532.p.clickup-attachments.com/t90131963532/fdfb5dc0-1328-4619-ad28-a0d809c791dc/image.png)

---

## 2. Crear una red Docker

Se crea una red para permitir la comunicación entre múltiples contenedores.

```bash
docker network create red-backend
```

![Crear red Docker](https://t90131963532.p.clickup-attachments.com/t90131963532/b4cde816-44e7-45b8-8bb1-445d0ed4ecae/image.png)

---

## 3. Crear el contenedor MySQL

Se crea un contenedor MySQL asociado a la red creada anteriormente.

```bash
docker run -d \
--name mysql-dev \
--network red-backend \
-e MYSQL_ROOT_PASSWORD=admindocker \
-p 3305:3306 \
mysql:8.0.39-debian
```

![Contenedor MySQL](https://t90131963532.p.clickup-attachments.com/t90131963532/2c7f9093-e925-49f1-921f-9740d1a18aa1/image.png)
![MySQL ejecutándose](https://t90131963532.p.clickup-attachments.com/t90131963532/2a0e7e33-1371-4297-a6b3-c0bb8e9979e1/image.png)

---

## 4. Ingresar a MySQL

Se accede al contenedor MySQL para trabajar directamente con la base de datos.

```bash
docker exec -it mysql-dev mysql -u root -p
```

Contraseña:
```
admindocker
```

![Ingreso a MySQL](https://t90131963532.p.clickup-attachments.com/t90131963532/bc21cbe9-37ef-4530-9ab1-83b76cdabf17/image.png)

---

## 5. Crear la base de datos

Dentro de MySQL se crea la base de datos que usará la aplicación.

```sql
CREATE DATABASE bdventas;
```

![Creación de base de datos](https://t90131963532.p.clickup-attachments.com/t90131963532/31151500-38aa-45be-87d2-ba36d35821c1/image.png)

---

## 6. Crear proyecto en Spring Initializr

Se genera el proyecto Spring Boot desde **Spring Initializr** con las dependencias necesarias.

![Spring Initializr](https://t90131963532.p.clickup-attachments.com/t90131963532/49050f40-47ab-4cd0-85c4-d6ad2720ae54/image.png)

---

## 7. Configurar application.properties

Se configura la conexión a la base de datos MySQL.

```properties
spring.application.name=demo-rest-bd
spring.datasource.url=jdbc:mysql://localhost:3305/bdventas?useSSL=false
spring.datasource.username=root
spring.datasource.password=admindocker
```

---

## 8. Ejecutar la aplicación localmente

Se ejecuta el proyecto para validar la conexión con MySQL.

![Ejecución local](https://t90131963532.p.clickup-attachments.com/t90131963532/6cc420a7-edc6-4aba-a40f-e3ebe21bf4f6/image.png)

---

## 9. Crear contenedor para el backend

Se crea un segundo contenedor donde se alojará la aplicación Spring Boot.

```bash
docker run -it \
-p 8085:8080 \
--name backend-ventas \
--network red-backend \
eclipse-temurin:21-jdk-alpine sh
```

![Contenedor backend](https://t90131963532.p.clickup-attachments.com/t90131963532/76a27537-89d4-4b06-92c4-0147f23c1d60/image.png)

Listado de contenedores activos:

![Contenedores](https://t90131963532.p.clickup-attachments.com/t90131963532/be7eea2f-6074-4653-aeb0-bb7eceee9340/image.png)

---

## 10. Actualizar el sistema del contenedor

Dentro del contenedor backend se actualiza el sistema operativo.

```bash
apk update
```

![apk update](https://t90131963532.p.clickup-attachments.com/t90131963532/c1db123b-f150-41f2-b4bb-64f9a1d8d3a3/image.png)

---

## 11. Instalar paquetes necesarios

Se instalan herramientas básicas requeridas.

```bash
apk add curl unzip
```

![Instalación de paquetes](https://t90131963532.p.clickup-attachments.com/t90131963532/2239565d-57aa-49ae-836a-8b32816f422c/image.png)

---

## 12. Ajustar configuración para Docker

Para el JAR que se ejecutará en Docker, se debe usar:
- El **nombre del contenedor MySQL** como host
- El **puerto interno 3306**

Ejemplo:

```properties
spring.datasource.url=jdbc:mysql://mysql-dev:3306/bdventas?useSSL=false
```

![Cambio de configuración](https://t90131963532.p.clickup-attachments.com/t90131963532/9d4db2e5-e2a2-420f-a7ba-ff1152b040a1/image.png)
![Configuración correcta](https://t90131963532.p.clickup-attachments.com/t90131963532/d9c4c1d7-714d-477a-9f21-c9ae5e095824/image.png)

---

## 13. Crear el JAR del proyecto

Desde IntelliJ IDEA se genera el archivo JAR.

```bash
mvn clean install -DskipTests
```

![Build JAR](https://t90131963532.p.clickup-attachments.com/t90131963532/cbd9c57a-03da-473b-abd6-13eb9fc7dd46/image.png)

---

## 14. Verificar el JAR generado

Se valida el archivo generado dentro de la carpeta `target`.

![JAR generado](https://t90131963532.p.clickup-attachments.com/t90131963532/e6b4b6aa-c963-4c97-9f4c-0c4cd133abec/image.png)

Se cambia a un nombre más corto:

![Renombrar JAR](https://t90131963532.p.clickup-attachments.com/t90131963532/4c723b11-8c1b-4ebd-b230-5241a1e34d7c/image.png)

---

## 15. Copiar el JAR al contenedor backend

Desde la terminal de IntelliJ:

```bash
docker cp ./target/app.jar backend-ventas:/app.jar
```

![Copiar JAR](https://t90131963532.p.clickup-attachments.com/t90131963532/df2d07c1-d66d-42b7-82ef-743f6ae34b0f/image.png)
![JAR copiado](https://t90131963532.p.clickup-attachments.com/t90131963532/90e1ae67-fda2-4d25-9e35-2d4b2475cf59/image.png)

---

## 16. Verificar el archivo en el contenedor

Dentro del contenedor se lista el contenido.

```bash
ls
```

![Verificación JAR](https://t90131963532.p.clickup-attachments.com/t90131963532/dd82302f-a8b1-4f3e-ad29-03628169a81c/image.png)

---

## 17. Ejecutar la aplicación

Finalmente se ejecuta el JAR dentro del contenedor.

```bash
java -jar /app.jar
```

![Aplicación ejecutándose](https://t90131963532.p.clickup-attachments.com/t90131963532/9eafd731-18b5-4aba-9105-b39e032542cd/image.png)

---

## Resultado final

- MySQL y backend corren en contenedores separados
- Ambos se comunican mediante una red Docker
- La aplicación Spring Boot se ejecuta correctamente usando MySQL

Procedimiento completo y validado.

