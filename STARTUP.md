Buka Docker Desktop → tunggu status "Engine running"
Buka terminal Ubuntu → wsl -d Ubuntu dari PowerShell
Baca dulu apa yang sudah nyala → docker ps
Lalu buka http://localhost:8081 (Airflow) & http://localhost:8100 (Spark)

Opsional (sekali saja) — stop container lama menghantui tiap sesi:
docker update --restart=no n8n-n8n-1 airflow_scheduler airflow_webserver

Cara buat docs/STARTUP.md di repo
==================================================

Melihat data :
cd ~/projects/dibimbing_spark_airflow
pwd