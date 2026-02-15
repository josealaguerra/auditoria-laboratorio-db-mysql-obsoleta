# Auditoría Laboratorio DB MySQL Obsoleta

Este proyecto despliega un entorno de laboratorio para simular una base de datos MySQL vulnerable, utilizando una versión obsoleta (MySQL 5.7, que ha alcanzado su "End of Life"). El objetivo es demostrar vulnerabilidades comunes en configuraciones inseguras de bases de datos, como contraseñas débiles, exposición de puertos estándar y falta de cifrado en reposo.

## Requisitos Previos

Antes de comenzar, asegúrate de tener instalados los siguientes componentes en tu sistema:

- **Docker**: Versión 20.10 o superior.
- **Docker Compose**: Versión 1.29 o superior.
- **Nmap** (opcional): Para pruebas de escaneo de puertos.
- **Cliente MySQL**: Para conectarte a la base de datos (puedes usar el cliente de línea de comandos o herramientas como MySQL Workbench).

Puedes verificar las versiones instaladas con:
```bash
docker --version
docker-compose --version
```

## Instalación y Despliegue

1. Clona o descarga este repositorio en tu máquina local.
2. Navega al directorio del proyecto:
   ```bash
   cd auditoria-laboratorio-db-mysql-obsoleta
   ```
3. Ejecuta el siguiente comando para levantar el contenedor en segundo plano:
   ```bash
   docker-compose up -d
   ```

Esto descargará la imagen de MySQL 5.7 y configurará el contenedor con las vulnerabilidades intencionales.

> **Nota**: Si encuentras un error de puerto ocupado (puerto 3306), detén cualquier otro servicio que lo esté usando o modifica el puerto en `docker-compose.yml`.

## Guía de Verificación (Replicando la Auditoría)

Actúa como auditor para confirmar que el entorno cumple con las condiciones de vulnerabilidad. A continuación, se detallan pruebas para verificar las configuraciones inseguras.

### Prueba A: Verificar Puerto Estándar y Versión (Nmap)

Escanea el puerto 3306 para confirmar la exposición del servicio:

```bash
nmap -sV -p 3306 localhost
```

**Resultado esperado**: El puerto 3306/tcp estará abierto y mostrará la versión MySQL 5.7.x, confirmando la exposición del servicio estándar.

### Prueba B: Verificar Acceso Root y Credenciales Débiles

Intenta conectarte a la base de datos usando las credenciales débiles:

```bash
mysql -h localhost -u root -p
```

Cuando se solicite la contraseña, ingresa: `root`

**Resultado esperado**: Acceso exitoso al shell de MySQL (`mysql>`). Esto confirma que la cuenta root es accesible remotamente con una contraseña trivial.

Para el usuario adicional:
```bash
mysql -h localhost -u admin_remoto -p
```

Contraseña: `password123`

### Prueba C: Verificar Falta de Cifrado en Reposo (TDE)

Simula el acceso físico al servidor inspeccionando los archivos de datos.

1. Navega a la carpeta `mysql_data` creada en el directorio del proyecto.
2. Entra en `mysql_data/ferreteria_db`.
3. Abre un archivo con extensión `.ibd` (por ejemplo, `db.opt` o archivos de tablas) con un editor de texto o hexadecimal.
4. Observa que los datos no están cifrados; podrías ver cadenas legibles si se insertaron datos de texto.

**Evidencia técnica**:
Conéctate a MySQL (como en la Prueba B) y ejecuta:

```sql
SHOW VARIABLES LIKE 'have_openssl';
SHOW VARIABLES LIKE 'innodb_encrypt_tables';
```

**Resultado esperado**: `innodb_encrypt_tables` estará en `OFF`, confirmando la falta de cifrado en reposo.

## Datos de Prueba (Opcional)

Para añadir realismo, inserta datos sensibles una vez conectado:

```sql
USE ferreteria_db;
CREATE TABLE tarjetas_credito (id INT, titular VARCHAR(100), numero VARCHAR(20));
INSERT INTO tarjetas_credito VALUES (1, 'Juan Perez', '4500-1234-5678-9010');
```

Esto permite simular capturas de tráfico con herramientas como Wireshark para observar datos en texto plano.

## Limpieza

Cuando termines las pruebas, detén y elimina el contenedor:

```bash
docker-compose down
```

> **Nota**: La carpeta `mysql_data` permanecerá en tu disco, simulando la persistencia de datos no cifrados.

## Contribuciones

Si deseas contribuir a este proyecto, por favor abre un issue o envía un pull request en el repositorio.

## Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo `LICENSE` para más detalles.