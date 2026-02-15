# auditoria-laboratorio-db-mysql-obsoleta
Vamos a desplegar una versión de MySQL (versión 5.7, que ya está "End of Life", lo que agrega una vulnerabilidad real de obsolescencia) configurada intencionalmente de forma insegura.

# 4. Despliegue del Escenario
Abre tu terminal (PowerShell, CMD o Terminal), navega hasta la carpeta laboratorio-auditoria y ejecuta:

Bash
docker-compose up -d

Esto descargará la imagen y levantará el contenedor en segundo plano.

5. Guía de Verificación (Replicando la Auditoría)
Ahora actuarás como el auditor para confirmar que el entorno cumple con las condiciones de la Cédula HALL-01.

Prueba A: Verificar Puerto Estándar y Versión (Nmap)
Si tienes Nmap instalado (o desde tu Kali Linux), ejecuta:

Bash
nmap -sV -p 3306 localhost
Resultado esperado: Verás el puerto 3306/tcp open y la versión MySQL 5.7.x, confirmando la exposición del servicio estándar.

Prueba B: Verificar Acceso Root y Credenciales Débiles
Intenta conectarte desde tu máquina "atacante" (tu PC o una VM):

Bash
mysql -h localhost -u root -p
Cuando te pida contraseña, escribe: root

Resultado esperado: Acceso exitoso al shell de MySQL (mysql>). Esto confirma que la cuenta root es accesible remotamente (o al menos desde fuera del contenedor) y usa una contraseña trivial.

Prueba C: Verificar Falta de Cifrado en Reposo (TDE)
Para esta prueba, simularemos el acceso físico al servidor.

En tu carpeta laboratorio-auditoria, verás que se creó una carpeta llamada mysql_data.

Entra en mysql_data/ferreteria_db.

Verás archivos con extensión .ibd (ej. db.opt o tablas que crees).

Abre uno de esos archivos con un editor de texto o hexadecimal. Aunque verás símbolos extraños, si insertaras datos de texto, podrías encontrar cadenas legibles o patrones claros.

Evidencia Técnica:
Para confirmar técnicamente que TDE (Transparent Data Encryption) está apagado, entra a MySQL (paso B) y ejecuta:

SQL
SHOW VARIABLES LIKE 'have_openssl';
SHOW VARIABLES LIKE 'innodb_encrypt_tables';
Resultado esperado: innodb_encrypt_tables estará en OFF. Esto es la evidencia para tu cédula de "Datos no cifrados en reposo".

6. Datos de Prueba (Opcional)
Para darle realismo, puedes insertar datos "sensibles" una vez conectado:

SQL
USE ferreteria_db;
CREATE TABLE tarjetas_credito (id INT, titular VARCHAR(100), numero VARCHAR(20));
INSERT INTO tarjetas_credito VALUES (1, 'Juan Perez', '4500-1234-5678-9010');
Esto te servirá si luego usas herramientas como Wireshark para capturar el tráfico y ver estos datos pasar en texto plano (otra vulnerabilidad).

¿Cómo detener el laboratorio?
Cuando termines tus pruebas, ejecuta:

Bash
docker-compose down
(Nota: La carpeta mysql_data quedará en tu disco con la información, simulando la persistencia de datos).