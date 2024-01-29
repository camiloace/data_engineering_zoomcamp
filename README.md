services:
  postgres:
    image: postgres:13
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=root
      - POSTGRES_DB=ny_taxi
    volumes:
      - postgres-db-volume:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "airflow"]
      interval: 5s
      retries: 5
    restart: always
"""
winpty docker run -it docker run -it \ 
  -e POSTGRES_USER = "root" \
  -e POSTGRES_PASSWORD = "root" \
  -e POSTGRES_DB = "ny_taxi" \
  -v c:/Users/USUARIO/data-engineering-zoomcamp/week_1_basics_n_setup/2_docekr_sql/ny_taxi_postgres-db-volume:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13

winpty docker run -it \ 
          -e POSTGRES_USER=root \
          -e POSTGRES_PASSWORD=root \
          -e POSTGRES_DB=ny_taxi \
          -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
          -p 5432:5432 \
          postgres:13
"""

winpty 
# Run postgres
docker run -it \
  -e POSTGRES_USER=root \
  -e POSTGRES_PASSWORD=root \
  -e POSTGRES_DB=ny_taxi \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13

#Run pgadmin
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  dpage/pgadmin4

#Network

docker network create pg-network

docker run -it \
  -e POSTGRES_USER=root \
  -e POSTGRES_PASSWORD=root \
  -e POSTGRES_DB=ny_taxi \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database1 \
  postgres:13

docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin \
  dpage/pgadmin4

#download dataset
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

URL="https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2021-01.parquet"
python ingest_data.py
  --user=root \
  --password=root \
  --host=localhost \
  --port=5432 \
  --db=ny_taxi \
  --table_name=yellow_taxi_trips \
  --url=${URL}
  
export URL="https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2021-01.parquet"
python ingest_data.py --user=root --password=root --host=localhost --port=5432 --db=ny_taxi --table_name=yellow_taxi_trips --url=${URL}

#build docker file with new parameter

docker build -t taxi_ingest:v001 . #versionado

#dockerizing

docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
  --user=root \
  --password=root \
  --host=pg-database \
  --port=5432 \
  --db=ny_taxi \
  --table_name=yellow_taxi_trips \
  --url=${URL}