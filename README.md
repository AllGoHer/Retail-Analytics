# Retail-Analytics

El proyecto consiste en el analisis de datos en el sector minorista, el cual se obtendrá los datos en bruto y serán procesados en sus tres instancias (Bronze, Silver and Gold), luego se transformará en un panel visual de Power BI con los resultados finales. Este presentará los ingresos finales de la empresa para observar su crecimiento y la estacionalidad de las ventas, la cual le permitirá controlar sus inventarios por regiones y replantear sus estrategias de ventas. 

![image](https://github.com/user-attachments/assets/80b0abe8-71e0-4b07-aeae-a0f2ea6023a2)

**Paso 1**: En tu terminal de VSCode, ve a la carpeta donde estás trabajando (ej: C:\Users\User\Reatil-PySpark).

Crea una carpeta nueva para este proyecto.

bash:

      mkdir Retail-PySpark

bash:

      cd Retail-PySpark

**Paso 2:** Crear el archivo "Receta" (docker-compose.yml) en Visual Studio Code.

Dentro de la carpeta Retail-PySpark, vas a crear un archivo llamado exactamente docker-compose.yml (sin espacios, todo en minúsculas).

Copia y pega el siguiente código dentro de ese archivo.

código:

        version: "3.8"

        services:
          spark-master:
            image: apache/spark:3.5.1
            container_name: spark-master
            hostname: spark-master
            command: >
              bash -c "
              /opt/spark/sbin/start-master.sh &&
              tail -f /opt/spark/logs/spark--org.apache.spark.deploy.master*.out
              "
            ports:
              - "8080:8080"   # Spark Master UI
              - "7077:7077"   # Spark Master port
            volumes:
              - ./app:/opt/spark-app
              - ./data:/opt/spark-data

          spark-worker-1:
            image: apache/spark:3.5.1
            container_name: spark-worker-1
            hostname: spark-worker-1
            command: >
              bash -c "/opt/spark/sbin/start-worker.sh spark://spark-master:7077 --cores 2 --memory 2g &&
              tail -f /opt/spark/logs/spark--org.apache.spark.deploy.worker*.out"
            ports:
              - "8081:8081"
              - "4040:4040"
            depends_on:
              - spark-master
            volumes:
              - ./app:/opt/spark-app
              - ./data:/opt/spark-data

          spark-worker-2:
            image: apache/spark:3.5.1
            container_name: spark-worker-2
            hostname: spark-worker-2
            command: >
              bash -c "/opt/spark/sbin/start-worker.sh spark://spark-master:7077 --cores 2 --memory 2g &&
              tail -f /opt/spark/logs/spark--org.apache.spark.deploy.worker*.out"
            ports:
              - "8082:8081"
            depends_on:
              - spark-master
            volumes:
              - ./app:/opt/spark-app
              - ./data:/opt/spark-data

 
Ahora abre tu Docker Desktop.

**Paso 3: ¡Ejecutar la magia!**

Ahora, asegúrate de que tu terminal esté dentro de la carpeta Retail-PySpark (la ruta debería decir algo como (venv) PS C:\Users\User\ Retail-PySpark >).

Ejecuta este comando:

                      docker-compose up -d

(El -d es para que corra en segundo plano, igual que cuando levantamos n8n).
