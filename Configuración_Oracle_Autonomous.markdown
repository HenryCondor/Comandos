# Configuración para Conexión a Oracle Cloud Autonomous en Codespaces

**Autor:** Henry Lunazco  
**Fecha:** Mayo 23, 2025

## Introducción
Este documento detalla los pasos realizados para configurar y conectar exitosamente a una base de datos Oracle Cloud Autonomous desde un entorno Codespaces, utilizando Python con la librería `oracledb`.

## Instalación y Configuración

### Instalación del Oracle Instant Client
Se instaló el cliente Oracle Instant Client versión 21.1:
- Descargar el paquete desde el sitio oficial de Oracle.
- Extraer en `/opt/oracle/instantclient_21_1`.
- Configurar la variable de entorno:
  ```
  export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_1:$LD_LIBRARY_PATH
  ```
- Verificar instalación:
  ```
  ls /opt/oracle/instantclient_21_1
  echo $LD_LIBRARY_PATH
  ```

### Instalación de Dependencias
Se instaló la dependencia requerida `libaio1` para el cliente Oracle:
```
sudo apt-get update
sudo apt-get install -y libaio1
```
Verificando instalación:
```
ldconfig -p | grep libaio
dpkg -l | grep libaio1
```

### Configuración del Entorno
Configurando el directorio del wallet y asegurando permisos:
```
chmod -R 755 /workspaces/AS232S4_PII_T09-be/Backend/Wallet_ECPPP
chmod -R 644 /workspaces/AS232S4_PII_T09-be/Backend/Wallet_ECPPP/*
```

### Configuración del Wallet
Subiendo y descomprimiendo el wallet descargado de Oracle Cloud:
```
unzip Wallet_ECPPP.zip -d /workspaces/AS232S4_PII_T09-be/Backend/Wallet_ECPPP
```
Archivos incluidos en el wallet:
- `truststore.jks`
- `sqlnet.ora`
- `tnsnames.ora`
- `cwallet.sso`
- `README`
- `ojdbc.properties`
- `ewallet.pem`
- `keystore.jks`
- `ewallet.p12`

Contenido típico de `sqlnet.ora`:
```
WALLET_LOCATION = (SOURCE = (METHOD = FILE) (METHOD_DATA = (DIRECTORY = /workspaces/AS232S4_PII_T09-be/Backend/Wallet_ECPPP)))
SSL_SERVER_DN_MATCH = yes
```

### Instalación de Python y Dependencias
Creando entorno virtual y instalando `oracledb`:
```
python -m venv venv
source venv/bin/activate
pip install oracledb
```

## Código Final
El script Python final utilizado para la conexión (`config.py`):

```python
import oracledb
import os

def get_db_connection():
    wallet_dir = "/workspaces/AS232S4_PII_T09-be/Backend/Wallet_ECPPP"
    if not os.path.exists(wallet_dir):
        return None
    oracledb.init_oracle_client(config_dir=wallet_dir)
    dsn = "ecppp_medium"
    try:
        connection = oracledb.connect(
            user="develop",
            password="PracticasPreProf25",
            dsn=dsn,
            config_dir=wallet_dir,
            ssl_server_dn_match=True
        )
        return connection
    except oracledb.Error:
        return None

def close_connection(connection):
    if connection:
        connection.close()

if __name__ == "__main__":
    connection = get_db_connection()
    if connection:
        close_connection(connection)
        print("¡Conexión exitosa a la base de datos Oracle Cloud Autonomous!")
    else:
        print("Error de conexión a la base de datos")
```

## Notas Finales
- Asegurar que la IP de Codespaces esté en la lista de control de acceso (ACL) de Oracle Cloud.
- Verificar el estado de la base de datos en la consola de Oracle Cloud.
- En caso de errores persistentes, revisar permisos y descargar un nuevo wallet.