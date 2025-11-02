


# 🎬 MovieLens Graph Analytics with PySpark + Neo4j

Dự án này xây dựng quy trình **ETL → Data Lake → Data Warehouse → Neo4j Graph** cho dữ liệu phim.  
Mục tiêu là tạo **Movie Knowledge Graph** phục vụ **Recommendation System** và **Graph Analytics**.

---

## 🏗️ Kiến trúc tổng quát

```

JSON Raw → PySpark ETL → Data Lake (CSV) → Data Warehouse (Parquet) → Neo4j Graph Database

````

Các thành phần:

| Component | Công nghệ | Chức năng |
|----------|----------|-----------|
| Data Processing | **PySpark + Jupyter Notebook** | Làm sạch và chuẩn hóa dữ liệu |
| Storage (Lake + Warehouse) | **Local Parquet** | Lưu dữ liệu cuối sau ETL |
| Graph Database | **Neo4j + APOC + GDS** | Lưu Movie Graph và chạy Recommendation |

---

## 🚀 1. Chuẩn bị môi trường

Yêu cầu trước khi chạy:

- Docker & Docker Compose
- RAM tối thiểu **6GB** (khuyến nghị 8GB)

Kiểm tra Docker:

```sh
docker --version
docker compose version
````

---

## 🐳 2. Chạy môi trường Docker

Chạy các dịch vụ:

```sh
docker compose up -d
```

Kiểm tra container:

```sh
docker ps
```

Bạn sẽ thấy:

| Service          | URL                                            | Ghi chú           |
| ---------------- | ---------------------------------------------- | ----------------- |
| Jupyter Notebook | [http://localhost:8888](http://localhost:8888) | Dùng để chạy ETL  |
| Neo4j Browser    | [http://localhost:7474](http://localhost:7474) | Truy vấn đồ thị   |
| Bolt driver      | `bolt://neo4j_db:7687`                         | Dùng trong Python |

### Đăng nhập Neo4j

| User  | Password         |
| ----- | ---------------- |
| neo4j | mysecretpassword |

---

## 📂 3. Cấu trúc thư mục

```
project/
│ docker-compose.yml
│ README.md
│
├─ notebooks/
│   ├─ 01_extract_raw_to_datalake.ipynb
│   ├─ 02_etl_pyspark.ipynb
│   └─ 03_load_to_neo4j.ipynb
│
├─ data/
│   ├─ raw/           ← chứa JSON gốc
│   ├─ datalake/      ← CSV sau chuyển đổi
│   ├─ warehouse/     ← Parquet sau làm sạch
│   └─ neo4j/         ← dữ liệu graph persistent
│
└─ neo4j/
    └─ init/          ← scripts khởi tạo DB
```

> **Lưu ý:** File `.gitignore` đã được cấu hình để không push dữ liệu thật lên GitHub.
> Bạn chỉ push **mã nguồn + notebooks**.

---

## 🧪 4. Chạy ETL Pipeline

### Bước 1: Mở Jupyter Notebook

```sh
docker logs pyspark_jupyter 2>&1 | grep token
```

Truy cập:
👉 `http://localhost:8888/?token=...`

---

### Bước 2: Extract → Data Lake

Chạy notebook:

```
notebooks/01_extract_raw_to_datalake.ipynb
```

Output sẽ nằm trong `data/datalake/`

---

### Bước 3: Transform → Data Warehouse

Chạy:

```
notebooks/02_etl_pyspark.ipynb
```

Dữ liệu sạch được lưu tại:

```
data/warehouse/
```

---

### Bước 4: Load vào Neo4j Graph

Chạy:

```
notebooks/03_load_to_neo4j.ipynb
```

Mặc định sẽ load **10,000 mẫu mỗi bảng** để đảm bảo hiệu năng trong Docker.

---

## 🔍 5. Kiểm tra dữ liệu trong Neo4j Browser

Truy cập:

👉 [http://localhost:7474](http://localhost:7474)

Chạy truy vấn:

```cypher
MATCH (n) RETURN labels(n), count(n);
```

Kiểm tra graph:

```cypher
MATCH (m:Movie)-[:HAS_TAG]->(t:Tag)
RETURN m.title, collect(t.tag_name)[0..5]
LIMIT 20;
```

---

## ⚡ 6. Tắt hệ thống khi xong

```sh
docker compose down
```

Nếu muốn giữ DB và dữ liệu:

✅ Không làm gì — vì bạn đã mount volume `./data/neo4j:/data`.

---

## 🧭 Roadmap mở rộng (Tiếp theo)

| Mục tiêu                            | Công nghệ                     | Trạng thái   |
| ----------------------------------- | ----------------------------- | ------------ |
| Graph Recommendation (User → Movie) | Neo4j GDS                     | 🚧 Next Step |
| Movie Similarity Search             | Embedding + Cosine Similarity | 🚧 Next Step |
| GNN Graph Neural Network            | PyTorch Geometric / DGL       | 🚧 Optional  |

---

