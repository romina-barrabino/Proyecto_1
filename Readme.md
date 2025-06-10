##Proyecto_parte_1
#Ingreso a la pagina web https://home.openweathermap.org/, me registro y creo una API llamada RomiAPI
#Instalo SQL Microsoft Server y creo una carpeta llamada weather_db
#Creo una carpeta en la computadora llamada Proyecto_1_2025 y la abro en el visual studio
#Descargo git desde https://git-scm.com/downloads/win
#Verifico la version de git 
git --version
#Para clonar use https debido a un problema de configuracion con SSH
git clone https://github.com/franncardenas/weather_project.git
#Instalo las librerias de requirements.txt usando la ruta del archivo
pip install -r C:\Users\PC\Desktop\Proyecto_1_2025\weather_project\requirements.txt
#En la carpeta extract_transform_load.py escribir el script de extraccion y carga en la base de datos:

#Librerias
import os
import requests
import pyodbc

#Parámetros
API_KEY = '19da0f4b67274245671f080a277d4972'
COUNTRY = 'Argentina'
SQL_SERVER_CONFIG = {
    'DRIVER': '{ODBC Driver 17 for SQL Server}',
    'SERVER': 'localhost',
    'DATABASE': 'weather_db',
    'UID': 'sa',
    'PWD': 'simpleplan1994'
}

#Extracción de datos
def extraer_datos(api_key, country):
    url = f'https://api.openweathermap.org/data/2.5/weather?q={country}&appid={api_key}&units=metric'
    response = requests.get(url)
    data = response.json()
    print("Respuesta de la API:", data)
    return data

#Transformación de datos
def transformar_datos(data):
    try:
        return {
            'country': data['sys']['country'],
            'temperature': data['main']['temp'],
            'humidity': data['main']['humidity']
        }
    except KeyError as e:
        raise Exception(f"Se produjo un error en: {e}")
print ("Se transformaron los datos correctamente")

#Carga de datos
def cargar_datos(datos, config):
    try:
        print(" Preparando cadena de conexión...")
        conn_str = (
            f"DRIVER={config['DRIVER']};"
            f"SERVER={config['SERVER']};"
            f"DATABASE={config['DATABASE']};"
            f"UID={config['UID']};"
            f"PWD={config['PWD']};"
        )
        print(f" Intentando conectar a: {conn_str}")
        conn = pyodbc.connect(conn_str)
        cursor = conn.cursor()
        print(" Conexión establecida con el servidor.")

        cursor.execute("""
            SELECT COUNT(*) 
            FROM INFORMATION_SCHEMA.TABLES 
            WHERE TABLE_NAME = 'WeatherData'
        """)
        table_exists = cursor.fetchone()[0]

        if table_exists == 0:
            print("La tabla no existe. Creándola...")
            cursor.execute("""
                CREATE TABLE WeatherData (
                    id INT IDENTITY(1,1) PRIMARY KEY,
                    country VARCHAR(50),
                    temperature FLOAT,
                    humidity INT,
                    fecha_registro DATETIME DEFAULT GETDATE()
                )
            """)
            conn.commit()
            print(" Tabla creada.")

        insert_query = """
            INSERT INTO WeatherData (country, temperature, humidity)
            VALUES (?, ?, ?)
        """
        cursor.execute(insert_query, (
            datos['country'],
            datos['temperature'],
            datos['humidity']
        ))
        conn.commit()
        print(" Datos insertados.")

    except Exception as e:
        import traceback
        print(" Error durante la carga de datos:")
        traceback.print_exc()

    finally:
        cursor.close()
        conn.close()
        print(" Conexión cerrada.")

#Proceso ETL completo
def ejecutar_etl():
    datos_extraidos = extraer_datos(API_KEY,COUNTRY)
    datos_transformados = transformar_datos(datos_extraidos)
    cargar_datos(datos_transformados, SQL_SERVER_CONFIG)
    print ("Se realizo el proceso ETL correctamente")

if __name__ == '__main__':
    ejecutar_etl()

#Ejecuto el script anterior y verifico la creacion de la tabla en SQL
#Ejecuto una consulta en SQL a modo de verificación
#Consulta 1: select * from dbo.weatherData
#Consulta 2: select country from weatherData

##Proyecto_parte_2 (Esto es lo que pude hacer pero no pude terminar por problemas con la tarjeta en ña cuenta de Bigquery)
#En Simbolo del sistema de windows entro a un entorno virtual
.\venv\Scripts\activate
#Instalo Google BigQuery
pip install google-cloud-bigquery
#Verifico su instalacion en el visual studio (uso un archivo de prueba para la ejecucion con terminacion .py)
import google.cloud.bigquery
print("BigQuery client importado correctamente.")

#Proyecto_parte_2_opcional: (Realice el mismo trabajo de la primera parte del poryecto pero agregue los conocimientos de privacidad para las API y las contraseñas)
#Utilice la misma base de datos creada en SQL.
#En visual studio cree la misma carpeta un archivo llamado .env, a la cual le adjunte lo siguiente:

API_KEY=19da0f4b67274245671f080a277d4972
CITY=Buenos Aires
SQL_DRIVER={ODBC Driver 17 for SQL Server}
SQL_SERVER=localhost
SQL_DATABASE=weather_db
SQL_UID=sa
SQL_PWD=simpleplan1994

#Instale en Visual studio python dotenv para la privacidad de los datos 
pip install python-dotenv
#En la misma carpeta del proyecto en visual studio, cree un archivo llamado "opcion_2_alternativa.py" y le agregue el siguiente script:

#En la misma base de datos, creo la tabla WeatherData_City para CITY=Buenos Aires

#Libreria
import os
import requests
import pyodbc
from dotenv import load_dotenv

# Cargo las variables desde el archivo .env
load_dotenv()

# Llamo a los parametros desde el archivo .env
API_KEY = os.getenv('API_KEY')
CITY = os.getenv('CITY')

SQL_SERVER_CONFIG = {
    'DRIVER': os.getenv('SQL_DRIVER'),
    'SERVER': os.getenv('SQL_SERVER'),
    'DATABASE': os.getenv('SQL_DATABASE'),
    'UID': os.getenv('SQL_UID'),
    'PWD': os.getenv('SQL_PWD')
}

#Extracción de datos
def extraer_datos(api_key, country):
    url = f'https://api.openweathermap.org/data/2.5/weather?q={country}&appid={api_key}&units=metric'
    response = requests.get(url)
    data = response.json()
    print("Respuesta de la API:", data)
    return data

#Transformación de datos
def transformar_datos(data):
    try:
        return {
            'city': data['name'],
            'temperature': data['main']['temp'],
            'humidity': data['main']['humidity']
        }
    except KeyError as e:
        raise Exception(f"Se produjo un error en: {e}")
print ("Se transformaron los datos correctamente")

#Carga de datos
def cargar_datos(datos, config):
    try:
        print(" Preparando cadena de conexión...")
        conn_str = (
            f"DRIVER={config['DRIVER']};"
            f"SERVER={config['SERVER']};"
            f"DATABASE={config['DATABASE']};"
            f"UID={config['UID']};"
            f"PWD={config['PWD']};"
        )
        print(f" Intentando conectar a: {conn_str}")
        conn = pyodbc.connect(conn_str)
        cursor = conn.cursor()
        print(" Conexión establecida con el servidor.")

        cursor.execute("""
            SELECT COUNT(*) 
            FROM INFORMATION_SCHEMA.TABLES 
            WHERE TABLE_NAME = 'WeatherData_City'
        """)
        table_exists = cursor.fetchone()[0]

        if table_exists == 0:
            print("La tabla no existe. Creándola...")
            cursor.execute("""
                CREATE TABLE WeatherData_City (
                    id INT IDENTITY(1,1) PRIMARY KEY,
                    city VARCHAR(50),
                    temperature FLOAT,
                    humidity INT,
                    fecha_registro DATETIME DEFAULT GETDATE()
                )
            """)
            conn.commit()
            print(" Tabla creada.")

        insert_query = """
            INSERT INTO WeatherData_City (city, temperature, humidity)
            VALUES (?, ?, ?)
        """
        cursor.execute(insert_query, (
            datos['city'],
            datos['temperature'],
            datos['humidity']
        ))
        conn.commit()
        print(" Datos insertados.")

    except Exception as e:
        import traceback
        print(" Error durante la carga de datos:")
        traceback.print_exc()

    finally:
        cursor.close()
        conn.close()
        print(" Conexión cerrada.")

#Proceso ETL completo
def ejecutar_etl():
    datos_extraidos = extraer_datos(API_KEY,CITY)
    datos_transformados = transformar_datos(datos_extraidos)
    cargar_datos(datos_transformados, SQL_SERVER_CONFIG)
    print ("Se realizo el proceso ETL correctamente")

if __name__ == '__main__':
    ejecutar_etl()

#Ejecuto el script anterior y verifico la creacion de la tabla en SQL
#Ejecuto una consulta en SQL a modo de verificación
#Consulta 1: select * from dbo.weatherData_City
#Consulta 2: select city from weatherData_City
