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

# ETAPA FUENTE DE DATOS
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






