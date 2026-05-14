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

 

**Paso 3: ¡Ejecutar la magia!**

Ahora, asegúrate de que tu terminal esté dentro de la carpeta Retail-PySpark (la ruta debería decir algo como (venv) PS C:\Users\User\ Retail-PySpark >).

Ejecuta este comando:

                      docker-compose up -d

(El -d es para que corra en segundo plano).

**Paso 4: Verificar en Docker Desktop**

Ve a tu Docker Desktop y haz clic en la pestaña "Containers" (o "Containers / Apps").

![image](https://github.com/user-attachments/assets/1779b925-94d7-4644-bc3c-dfab24675def)

## ETAPA FUENTE DE DATOS
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________

Generamos un archivo de datos crudos (generate_raw_data.py)
Ahora en VSCode creamos el archivo genetarte_raw_data.py 

Código:

        import csv
        import random
        from datetime import datetime, timedelta
        import os

        # -------------------------------
        # Configuration
        # -------------------------------
        BASE_DIR = os.getcwd()            # Where PowerShell runs the script
        OUTPUT_DIR = os.path.join(BASE_DIR, "data/raw")
        OUTPUT_FILE = "retail_sales_raw.csv"
        NUM_RECORDS = 1_000_000

        os.makedirs(OUTPUT_DIR, exist_ok=True)
        file_path = os.path.join(OUTPUT_DIR, OUTPUT_FILE)

        # -------------------------------
        # Reference Data
        # -------------------------------
        cities = [
            ("New York", "NY"), ("Los Angeles", "CA"), ("Chicago", "IL"),
            ("Houston", "TX"), ("Phoenix", "AZ"), ("Philadelphia", "PA"),
            ("San Antonio", "TX"), ("San Diego", "CA"), ("Dallas", "TX"),
            ("San Jose", "CA")
        ]

        categories = {
            "Electronics": (100, 2000),
            "Fashion": (20, 500),
            "Grocery": (1, 50),
            "Furniture": (50, 1500),
            "Sports": (10, 800)
        }

        payment_types = ["Card", "UPI", "COD", "Crypto", None]
        genders = ["M", "F", "Male", "Female", None]
        order_statuses = ["Delivered", "Cancelled", "Returned"]

        start_date = datetime(2023, 1, 1)
        end_date = datetime(2026, 1, 1)

        # -------------------------------
        # Helper Functions
        # -------------------------------
        def random_date(start, end):
            delta = end - start
            return start + timedelta(days=random.randint(0, delta.days))

        # -------------------------------
        # Data Generation
        # -------------------------------
        with open(file_path, mode="w", newline="", encoding="utf-8") as file:
            writer = csv.writer(file)

            writer.writerow([
                "transaction_id",
                "order_date",
                "ship_date",
                "customer_id",
                "customer_age",
                "gender",
                "product_id",
                "product_category",
                "quantity",
                "unit_price",
                "discount_pct",
                "city",
                "state",
                "payment_type",
                "order_status",
                "ingestion_date"
            ])

            for i in range(NUM_RECORDS):
                transaction_id = random.randint(1, NUM_RECORDS // 2)
                order_date = random_date(start_date, end_date)
                ship_date = order_date + timedelta(days=random.randint(-3, 10))

                customer_id = f"CUST{random.randint(1, 200_000)}"
                customer_age = random.choice([
                    random.randint(18, 70),
                    random.randint(-10, 10),
                    random.randint(120, 200),
                    None
                ])

                gender = random.choice(genders)
                category = random.choice(list(categories.keys()))
                price_min, price_max = categories[category]

                unit_price = random.choice([
                    round(random.uniform(price_min, price_max), 2),
                    -random.uniform(1, 100),
                    None
                ])

                quantity = random.choice([
                    random.randint(1, 10),
                    0,
                    -random.randint(1, 5)
                ])

                discount_pct = random.choice([
                    round(random.uniform(0, 50), 2),
                    round(random.uniform(60, 150), 2),
                    None
                ])

                city, state = random.choice(cities)

                writer.writerow([
                    transaction_id,
                    order_date.strftime("%Y-%m-%d"),
                    ship_date.strftime("%Y-%m-%d"),
                    customer_id,
                    customer_age,
                    gender,
                    f"PROD{random.randint(1, 50_000)}",
                    category,
                    quantity,
                    unit_price,
                    discount_pct,
                    city,
                    state,
                    random.choice(payment_types),
                    random.choice(order_statuses),
                    datetime.now().strftime("%Y-%m-%d")
                ])

        print(f"✅ Generated {NUM_RECORDS} records at:\n{file_path}")


![image](https://github.com/user-attachments/assets/d52e9238-33ac-405e-a369-c5e805f6ee13)

Luego en consola ejecutamos el siguiente comando
Python generate_raw_data.py

![image](https://github.com/user-attachments/assets/a031c5cc-fff0-4e91-a614-57be6918d6c5)

El cual generará el archivo de datos con cual trabajaremos.

![image](https://github.com/user-attachments/assets/955d04ef-ae19-4fcc-aec0-a9a0bc987040)

Hacemos la chispa PySpark en consola.

Luego en la terminal:
 
                      docker exec -it spark-worker-1 /opt/spark/bin/pyspark --master spark://spark-master:7077

![image](https://github.com/user-attachments/assets/529f95af-7bac-4ba5-af89-d5defec8f8a9)

Ahora vamos a Spark UI:

![image](https://github.com/user-attachments/assets/9cb47184-9ba7-4964-8766-c740695b3dbb)

Luego vamos al otro puerto (localhost:4040)

![image](https://github.com/user-attachments/assets/3ef9089c-8788-4188-a3c1-6a814d55de87)

## ETAPA BRONCE (Raw Data)
____________________________________________________________________________________________________________________________________________________________________________________________________________________________

En la terminal escribimos el siguiente comando para importar las librerías

bash:

     from pyspark.sql.types import(StructType, StructField, IntegerType, StringType, DoubleType, DateType)

bash:

     bronze_schema = (StructType([
     StructField("transaction_id", IntegerType(), True),  
     StructField("order_date", DateType(), True), 
     StructField("ship_date", DateType(), True), 
     StructField("customer_id", StringType(), True), 
     StructField("customer_age", IntegerType(), True), 
     StructField("gender", StringType(), True), 
     StructField("product_id", StringType(), True),
     StructField("product_category", StringType(), True), 
     StructField("quantity", IntegerType(), True),
     StructField("unit_price", DoubleType(), True), 
     StructField("discount_pct", DoubleType(), True), 
     StructField("city", StringType(), True), 
     StructField("state", StringType(), True), 
     StructField("payment_type", StringType(), True), 
     StructField("order_status", StringType(), True), 
     StructField("ingestion_date", DateType(), True)]))


![image](https://github.com/user-attachments/assets/d88c5b33-2362-4bd3-ac47-b88f110def39)

Ahora creamos el DataFrame bronce.

bash:

     bronze_df = spark.read.option("header", "true").schema(bronze_schema).csv("/opt/spark-data/raw/retail_sales_raw.csv")


![image](https://github.com/user-attachments/assets/2bbe3480-5e6d-43af-a189-963b1115b695)

Luego para confirmar el proceso hacemos lo siguiente.

bash:

     bronze_df.count()


![image](https://github.com/user-attachments/assets/dd39b81f-0df5-4a43-a65e-732a7d3bb019)

Ahora imprimiremos el esquema del DataFrame

bash:

      bronze_df.printSchema()

      
![image](https://github.com/user-attachments/assets/996598f4-01a0-44ea-adfc-4afb236557b4)

Luego escribimos los datos en formato parquet

bash:

     bronze_df.write.mode("overwrite").parquet("/opt/spark-data/bronze/retail_sales_bronze.parquet")

![image](https://github.com/user-attachments/assets/1509aee1-29e6-4fed-927e-b6e8aa2e958c)

Para observar como proceso los datos vamos a Spark UI (localhost:4040)

![image](https://github.com/user-attachments/assets/468cf76d-ce2d-4b43-900d-7cc8186e6b04)

Realizo 4 particiones

![image](https://github.com/user-attachments/assets/5b12db63-de51-409a-851d-e89fcc833486)

![image](https://github.com/user-attachments/assets/e2439ab4-8bef-4af2-a643-7996ece62d7f)

## ETAPA SILVER (Cleaned Data)
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________

Importamos las librerías de funciones de PySpark 

bash:

      from pyspark.sql.functions import col, when, trim

bash:

      bronze_df = spark.read.parquet("/opt/spark-data/bronze/retail_sales_bronze.parquet")

Ahora agrupamos por transaction_id., contamos y filtramos por columnas mayores que 1, y solo mostramos los top 5 filas.

bash:

      bronze_df.groupby("transaction_id").count().filter(col("count") > 1).show(5, truncate=False)

![image](https://github.com/user-attachments/assets/c07a5e2f-ecc5-41a4-a765-8685649ac11b)

En este proceso eliminamos los duplicados por “transaction_id”

bash: 

     silver_df = bronze_df.dropDuplicates(["transaction_id"])

bash:

      silver_df.count()

![image](https://github.com/user-attachments/assets/a4481438-9c86-45dd-bbf9-fc0b8930d1eb)

•	Correcciones en fecha de envío.

Lo que hará es filtrar por columna de fecha de envío y que la fecha de envío sea menor a la fecha de pedidos y que muestre top 5

bash: 

      bronze_df.filter(col("ship_date") < col("order_date")).show(5)


![image](https://github.com/user-attachments/assets/8cecb16e-6a49-405b-b041-6e5a5358303a)

Ahora creamos otra columna en “silver_df”

Se creará una nueva columna, asignando las nuevas fechas de envío que ya están limpias. Para ello Nombrara la nueva columna como fecha de envío, cuando se llame a la fecha de envío se menor a la fecha de pedido solicitada, pues ahora cambiaremos esos datos y los remplazamos por None y los restos de datos deberán ser igual a los de la fecha de envío. 

bash: 

      silver_df = silver_df.withColumn("ship_date", when(col("ship_date") < col("order_date"), None).otherwise(col("ship_date")))

Luego tenemos que verificar que ninguno de los datos sean igual a 0

bash:

     silver_df.filter(col("quantity") <= 0).show(5)

![image](https://github.com/user-attachments/assets/50a24473-f6b8-4620-8c58-4a8c11e835c8)

Ahora limpiaremos estos datos

bash:

      silver_df = silver_df.filter(col("quantity") > 0)
      
Luego buscamos valores con precio unitarios falsos.

bash:

      silver_df.filter(col("unit_price") <= 0).show(5)


![image](https://github.com/user-attachments/assets/0626b656-2ba0-4913-8b69-602623036f93)

Ahora filtramos todos los precios unitarios con valores negativos, pero vamos a filtrarlos y asignarles valores nulos para no perder registro adicional.

bash:

      silver_df = silver_df.withColumn("unit_price", when(col("unit_price") <= 0, None).otherwise(col("unit_price")))

En el siguiente proceso toca ver el descuento que sean mayores que 100.

bash:

     silver_df.filter(col("discount_pct") > 100).show(5)

![image](https://github.com/user-attachments/assets/67d67130-7515-48bc-b307-5d9df7d3327f)

bash:

      silver_df = silver_df.withColumn("discount_pct", when((col("discount_pct") < 0) | (col("discount_pct") > 100), None).otherwise(col("discount_pct")))

      
Ahora filtramos por edad

bash:

      silver_df.filter((col("customer_age") < 15) | (col("customer_age") > 100)).show(5)


![image](https://github.com/user-attachments/assets/5cbf4578-8082-492c-b9d4-2823aed10d24)

bash:

       silver_df = silver_df.withColumn("customer_age", when((col("customer_age") < 15) | (col("customer_age") > 100), None).otherwise(col("customer_age")))

Ahora agrupamos por genero

bash:

     silver_df.groupBy("gender").count().show()


![image](https://github.com/user-attachments/assets/50747b14-3cea-4232-8b63-522708b9274c)

Luego tenemos designar solo femenino y masculino

bash:

      silver_df = silver_df.withColumn(
      "gender",
      when(upper(trim(col("gender"))) == "MALE", "M")
      .when(upper(trim(col("gender"))) == "FEMALE", "F")
      .when(col("gender").isin("M", "F"), col("gender"))
      .otherwise(None)
       )

Ahora vemos el tipo de pago y eliminar las criptomonedas o tipo de moneda Falso.

bash:

      silver_df.filter(~col("payment_type").isin("Card", "UPI", "COD")).show(5)


![image]()

![image]()

![image]()

![image]()

![image]()



